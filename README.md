# OASIS-INFOBYTE-INTERNSHIP-T4
A Java Swing-based Online Examination System developed as the final task of my Java Development Internship at Oasis Infobyte, featuring student login, profile management, timed MCQ exams, automatic submission, and result evaluation.
~~~code~~~~

import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.Arrays;

public class OnlineExaminationSystem extends JFrame {

    private final CardLayout cardLayout = new CardLayout();
    private final JPanel mainPanel = new JPanel(cardLayout);

    private String username = "student";
    private String password = "1234";
    private String displayName = "Student";

    private boolean examActive = false;
    private boolean submitted = false;

    private final String[] questions = {
            "Which keyword is used to inherit a class in Java?",
            "Which method is the entry point of a Java program?",
            "Which collection stores unique elements?",
            "Which keyword creates an object in Java?",
            "Which package contains Swing components?"
    };

    private final String[][] options = {
            {"implements", "extends", "inherits", "super"},
            {"start()", "run()", "main()", "init()"},
            {"ArrayList", "List", "HashSet", "LinkedList"},
            {"class", "new", "this", "void"},
            {"java.io", "java.util", "javax.swing", "java.sql"}
    };

    // Correct answers are option indexes (0-based).
    private final int[] correctAnswers = {1, 2, 2, 1, 2};
    private final int[] selectedAnswers = new int[questions.length];

    private int currentQuestion = 0;
    private int timeLeft = 30 * 60;
    private long startTime;
    private long endTime;

    private JLabel timerLabel;
    private JLabel questionLabel;
    private JLabel questionNumberLabel;
    private final JRadioButton[] radioButtons = new JRadioButton[4];
    private final ButtonGroup buttonGroup = new ButtonGroup();

    private Timer timer;

    private JTextField loginUserField;
    private JPasswordField loginPasswordField;

    private JTextField displayNameField;
    private JPasswordField newPasswordField;

    public OnlineExaminationSystem() {
        setTitle("Online Examination System");
        setSize(700, 500);
        setLocationRelativeTo(null);
        setDefaultCloseOperation(DO_NOTHING_ON_CLOSE);

        Arrays.fill(selectedAnswers, -1);

        mainPanel.add(createLoginPanel(), "LOGIN");
        mainPanel.add(createProfilePanel(), "PROFILE");
        mainPanel.add(createExamPanel(), "EXAM");

        add(mainPanel);

        addWindowListener(new WindowAdapter() {
            @Override
            public void windowClosing(WindowEvent e) {
                if (examActive) {
                    int choice = JOptionPane.showConfirmDialog(
                            OnlineExaminationSystem.this,
                            "Are you sure you want to quit the exam?",
                            "Confirm Exit",
                            JOptionPane.YES_NO_OPTION,
                            JOptionPane.WARNING_MESSAGE
                    );

                    if (choice == JOptionPane.YES_OPTION) {
                        if (timer != null) {
                            timer.stop();
                        }
                        examActive = false;
                        dispose();
                    }
                } else {
                    dispose();
                }
            }
        });

        cardLayout.show(mainPanel, "LOGIN");
        setVisible(true);
    }

    // ---------------- LOGIN SCREEN ----------------

    private JPanel createLoginPanel() {
        JPanel panel = new JPanel(new GridBagLayout());
        panel.setBorder(BorderFactory.createEmptyBorder(20, 30, 20, 30));

        JPanel form = new JPanel(new GridLayout(0, 2, 10, 15));
        form.setBorder(BorderFactory.createTitledBorder(
                "Student Login"
        ));

        loginUserField = new JTextField("student", 15);
        loginPasswordField = new JPasswordField("1234", 15);

        form.add(new JLabel("Username:"));
        form.add(loginUserField);

        form.add(new JLabel("Password:"));
        form.add(loginPasswordField);

        JButton loginButton = new JButton("Login");
        loginButton.addActionListener(e -> login());

        form.add(new JLabel());
        form.add(loginButton);

        panel.add(form);
        return panel;
    }

    private void login() {
        String enteredUser = loginUserField.getText().trim();
        String enteredPassword =
                new String(loginPasswordField.getPassword());

        if (enteredUser.equals(username)
                && enteredPassword.equals(password)) {

            cardLayout.show(mainPanel, "PROFILE");

        } else {
            JOptionPane.showMessageDialog(
                    this,
                    "Invalid username or password!",
                    "Login Failed",
                    JOptionPane.ERROR_MESSAGE
            );
        }
    }

    // ---------------- PROFILE UPDATE SCREEN ----------------

    private JPanel createProfilePanel() {
        JPanel panel = new JPanel(new GridBagLayout());

        JPanel form = new JPanel(new GridLayout(0, 2, 10, 15));
        form.setBorder(BorderFactory.createTitledBorder(
                "Update Profile Before Exam"
        ));

        displayNameField = new JTextField(displayName, 15);
        newPasswordField = new JPasswordField(password, 15);

        form.add(new JLabel("Display Name:"));
        form.add(displayNameField);

        form.add(new JLabel("New Password:"));
        form.add(newPasswordField);

        JButton startButton = new JButton("Save & Start Exam");
        startButton.addActionListener(e -> updateProfile());

        form.add(new JLabel());
        form.add(startButton);

        panel.add(form);
        return panel;
    }

    private void updateProfile() {
        String name = displayNameField.getText().trim();
        String newPass = new String(newPasswordField.getPassword());

        if (name.isEmpty() || newPass.isEmpty()) {
            JOptionPane.showMessageDialog(
                    this, "Name and password cannot be empty!"
            );
            return;
        }

        displayName = name;
        password = newPass;

        startExam();
    }

    // ---------------- EXAM SCREEN ----------------

    private JPanel createExamPanel() {
        JPanel panel = new JPanel(new BorderLayout());
        panel.setBorder(BorderFactory.createEmptyBorder(20, 25, 20, 25));

        JPanel topPanel = new JPanel(new BorderLayout());

        JLabel welcomeLabel = new JLabel(
                "Online Examination - " + displayName
        );

        timerLabel = new JLabel("Time Left: 30:00");
        timerLabel.setFont(new Font("Arial", Font.BOLD, 18));

        topPanel.add(welcomeLabel, BorderLayout.WEST);
        topPanel.add(timerLabel, BorderLayout.EAST);

        panel.add(topPanel, BorderLayout.NORTH);

        JPanel centerPanel = new JPanel();
        centerPanel.setLayout(new BoxLayout(
                centerPanel, BoxLayout.Y_AXIS
        ));

        questionNumberLabel = new JLabel();
        questionNumberLabel.setFont(
                new Font("Arial", Font.BOLD, 16)
        );

        questionLabel = new JLabel();
        questionLabel.setFont(new Font("Arial", Font.PLAIN, 16));

        centerPanel.add(questionNumberLabel);
        centerPanel.add(Box.createVerticalStrut(15));
        centerPanel.add(questionLabel);
        centerPanel.add(Box.createVerticalStrut(20));

        for (int i = 0; i < 4; i++) {
            radioButtons[i] = new JRadioButton();
            radioButtons[i].setFont(
                    new Font("Arial", Font.PLAIN, 14)
            );

            buttonGroup.add(radioButtons[i]);
            centerPanel.add(radioButtons[i]);
            centerPanel.add(Box.createVerticalStrut(10));
        }

        panel.add(centerPanel, BorderLayout.CENTER);

        JPanel bottomPanel = new JPanel(
                new FlowLayout(FlowLayout.CENTER, 15, 5)
        );

        JButton previousButton = new JButton("Previous");
        JButton nextButton = new JButton("Next");
        JButton submitButton = new JButton("Submit Exam");

        previousButton.addActionListener(e -> {
            saveAnswer();

            if (currentQuestion > 0) {
                currentQuestion--;
                showQuestion();
            }
        });

        nextButton.addActionListener(e -> {
            saveAnswer();

            if (currentQuestion < questions.length - 1) {
                currentQuestion++;
                showQuestion();
            }
        });

        submitButton.addActionListener(e -> {
            saveAnswer();

            int choice = JOptionPane.showConfirmDialog(
                    this,
                    "Are you sure you want to submit the exam?",
                    "Confirm Submission",
                    JOptionPane.YES_NO_OPTION
            );

            if (choice == JOptionPane.YES_OPTION) {
                submitExam();
            }
        });

        bottomPanel.add(previousButton);
        bottomPanel.add(nextButton);
        bottomPanel.add(submitButton);

        panel.add(bottomPanel, BorderLayout.SOUTH);

        return panel;
    }

    private void startExam() {
        currentQuestion = 0;
        timeLeft = 30 * 60;
        submitted = false;
        examActive = true;

        Arrays.fill(selectedAnswers, -1);

        startTime = System.currentTimeMillis();

        showQuestion();
        cardLayout.show(mainPanel, "EXAM");

        if (timer != null) {
            timer.stop();
        }

        timer = new Timer(1000, e -> {
            timeLeft--;
            updateTimer();

            if (timeLeft <= 0) {
                timer.stop();

                JOptionPane.showMessageDialog(
                        this,
                        "Time is up! Your exam will be submitted."
                );

                submitExam();
            }
        });

        timer.start();
    }

    private void showQuestion() {
        questionNumberLabel.setText(
                "Question " + (currentQuestion + 1)
                        + " of " + questions.length
        );

        questionLabel.setText(
                "<html><body style='width:500px'>"
                        + questions[currentQuestion]
                        + "</body></html>"
        );

        buttonGroup.clearSelection();

        for (int i = 0; i < 4; i++) {
            radioButtons[i].setText(options[currentQuestion][i]);

            if (selectedAnswers[currentQuestion] == i) {
                radioButtons[i].setSelected(true);
            }
        }
    }

    private void saveAnswer() {
        for (int i = 0; i < 4; i++) {
            if (radioButtons[i].isSelected()) {
                selectedAnswers[currentQuestion] = i;
                return;
            }
        }

        selectedAnswers[currentQuestion] = -1;
    }

    private void updateTimer() {
        int minutes = timeLeft / 60;
        int seconds = timeLeft % 60;

        timerLabel.setText(String.format(
                "Time Left: %02d:%02d", minutes, seconds
        ));
    }

    // ---------------- SUBMISSION & RESULTS ----------------

    private void submitExam() {
        if (submitted) {
            return;
        }

        saveAnswer();

        submitted = true;
        examActive = false;
        endTime = System.currentTimeMillis();

        if (timer != null) {
            timer.stop();
        }

        showResult();
    }

    private void showResult() {
        int score = 0;

        for (int i = 0; i < questions.length; i++) {
            if (selectedAnswers[i] == correctAnswers[i]) {
                score++;
            }
        }

        long elapsedSeconds =
                Math.max(0, (endTime - startTime) / 1000);

        elapsedSeconds = Math.min(
                elapsedSeconds, 30 * 60
        );

        int minutes = (int) (elapsedSeconds / 60);
        int seconds = (int) (elapsedSeconds % 60);

        JPanel resultPanel = new JPanel(new BorderLayout(10, 10));
        resultPanel.setBorder(
                BorderFactory.createEmptyBorder(15, 20, 15, 20)
        );

        JLabel heading = new JLabel(
                "Exam Result - " + displayName,
                SwingConstants.CENTER
        );

        heading.setFont(new Font("Arial", Font.BOLD, 22));

        JLabel scoreLabel = new JLabel(
                "Score: " + score + " out of " + questions.length
                        + "     |     Time Taken: "
                        + String.format("%02d:%02d", minutes, seconds),
                SwingConstants.CENTER
        );

        JPanel summary = new JPanel(new GridLayout(0, 1, 5, 5));
        summary.add(heading);
        summary.add(scoreLabel);

        resultPanel.add(summary, BorderLayout.NORTH);

        JTextArea breakdown = new JTextArea();
        breakdown.setEditable(false);
        breakdown.setFont(new Font("Monospaced", Font.PLAIN, 12));

        for (int i = 0; i < questions.length; i++) {
            breakdown.append(
                    "\nQ" + (i + 1) + ": " + questions[i] + "\n"
            );

            breakdown.append(
                    "Your answer: "
                            + (selectedAnswers[i] == -1
                            ? "Not answered"
                            : options[i][selectedAnswers[i]])
                            + "\n"
            );

            breakdown.append(
                    "Correct answer: "
                            + options[i][correctAnswers[i]] + "\n"
            );

            breakdown.append(
                    selectedAnswers[i] == correctAnswers[i]
                            ? "Result: CORRECT\n"
                            : "Result: INCORRECT\n"
            );

            breakdown.append("--------------------------------\n");
        }

        resultPanel.add(
                new JScrollPane(breakdown), BorderLayout.CENTER
        );

        JButton logoutButton = new JButton("Logout");

        logoutButton.addActionListener(e -> {
            loginUserField.setText(username);
            loginPasswordField.setText("");
            displayNameField.setText(displayName);
            newPasswordField.setText(password);

            mainPanel.add(resultPanel, "RESULT");
            cardLayout.show(mainPanel, "LOGIN");
        });

        JPanel bottom = new JPanel();
        bottom.add(logoutButton);

        resultPanel.add(bottom, BorderLayout.SOUTH);

        mainPanel.add(resultPanel, "RESULT");
        cardLayout.show(mainPanel, "RESULT");
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(
                OnlineExaminationSystem::new
        );
    }
}
