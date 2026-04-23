import javax.swing.*;
import javax.swing.border.EmptyBorder;
import java.awt.*;
import java.io.*;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.Comparator;

/**
 * Улучшенный планировщик задач
 */
public class Planirovschik {

    private JFrame frame;
    private DefaultListModel<String> listModel;
    private JList<String> taskList;
    private JTextField taskInput;
    private JTextField dateInput;
    private JLabel statusLabel;

    private static final DateTimeFormatter DATE_FORMAT =
            DateTimeFormatter.ofPattern("dd.MM.yyyy");

    private static final String FILE_NAME = "tasks.txt";

    public Planirovschik() {
        createGUI();
        loadTasks(); // автозагрузка
    }

    private void createGUI() {

        frame = new JFrame("📅 Планировщик задач PRO");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(600, 650);
        frame.setLocationRelativeTo(null);

        JPanel main = new JPanel(new BorderLayout(10, 10));
        main.setBorder(new EmptyBorder(15, 15, 15, 15));

        JLabel title = new JLabel("📋 Планировщик задач", JLabel.CENTER);
        title.setFont(new Font("Segoe UI", Font.BOLD, 24));
        main.add(title, BorderLayout.NORTH);

        listModel = new DefaultListModel<>();
        taskList = new JList<>(listModel);

        JScrollPane scroll = new JScrollPane(taskList);
        scroll.setBorder(BorderFactory.createTitledBorder("Список задач"));
        main.add(scroll, BorderLayout.CENTER);

        // ===== INPUT =====
        taskInput = new JTextField();
        dateInput = new JTextField();

        JPanel inputPanel = new JPanel(new GridLayout(4, 1, 5, 5));
        inputPanel.add(new JLabel("Задача:"));
        inputPanel.add(taskInput);
        inputPanel.add(new JLabel("Дата (ДД.ММ.ГГГГ):"));
        inputPanel.add(dateInput);

        // ===== BUTTONS =====
        JButton add = new JButton("Добавить");
        JButton done = new JButton("Выполнено");
        JButton delete = new JButton("Удалить");
        JButton save = new JButton("Сохранить");
        JButton clear = new JButton("Очистить");

        JPanel buttons = new JPanel(new FlowLayout());
        buttons.add(add);
        buttons.add(done);
        buttons.add(delete);
        buttons.add(save);
        buttons.add(clear);

        statusLabel = new JLabel("Готово к работе");

        JPanel bottom = new JPanel(new BorderLayout());
        bottom.add(inputPanel, BorderLayout.NORTH);
        bottom.add(buttons, BorderLayout.CENTER);
        bottom.add(statusLabel, BorderLayout.SOUTH);

        main.add(bottom, BorderLayout.SOUTH);

        // ===== ACTIONS =====
        add.addActionListener(e -> addTask());
        done.addActionListener(e -> markDone());
        delete.addActionListener(e -> deleteTask());
        save.addActionListener(e -> saveTasks());
        clear.addActionListener(e -> clearAll());

        frame.add(main);
        frame.setVisible(true);
    }

    // ===== ДОБАВЛЕНИЕ =====
    private void addTask() {
        String task = taskInput.getText().trim();
        String date = dateInput.getText().trim();

        if (task.isEmpty()) {
            statusLabel.setText("❌ Введите задачу");
            return;
        }

        String result;

        if (!date.isEmpty()) {
            try {
                LocalDate.parse(date, DATE_FORMAT);
                result = "📌 " + task + " | до: " + date;
            } catch (Exception e) {
                statusLabel.setText("❌ Неверная дата");
                return;
            }
        } else {
            result = "📌 " + task;
        }

        listModel.addElement(result);

        taskInput.setText("");
        dateInput.setText("");

        statusLabel.setText("✅ Добавлено");
    }

    // ===== ВЫПОЛНЕНО =====
    private void markDone() {
        int index = taskList.getSelectedIndex();

        if (index == -1) {
            statusLabel.setText("⚠️ Выберите задачу");
            return;
        }

        String task = listModel.get(index);

        if (!task.startsWith("✅")) {
            listModel.set(index, "✅ " + task);
        }

        sortTasks();
    }

    // ===== УДАЛЕНИЕ =====
    private void deleteTask() {
        int index = taskList.getSelectedIndex();

        if (index != -1) {
            listModel.remove(index);
        }
    }

    // ===== ОЧИСТКА =====
    private void clearAll() {
        listModel.clear();
    }

    // ===== СОХРАНЕНИЕ =====
    private void saveTasks() {

        try (BufferedWriter bw = new BufferedWriter(new FileWriter(FILE_NAME))) {

            for (int i = 0; i < listModel.size(); i++) {
                bw.write(listModel.get(i));
                bw.newLine();
            }

            statusLabel.setText("💾 Сохранено");

        } catch (IOException e) {
            statusLabel.setText("❌ Ошибка сохранения");
        }
    }

    // ===== ЗАГРУЗКА =====
    private void loadTasks() {

        File file = new File(FILE_NAME);
        if (!file.exists()) return;

        try (BufferedReader br = new BufferedReader(new FileReader(file))) {

            String line;
            while ((line = br.readLine()) != null) {
                listModel.addElement(line);
            }

        } catch (IOException e) {
            statusLabel.setText("❌ Ошибка загрузки");
        }
    }

    // ===== СОРТИРОВКА =====
    private void sortTasks() {

        ArrayList<String> tasks = new ArrayList<>();

        for (int i = 0; i < listModel.size(); i++) {
            tasks.add(listModel.get(i));
        }

        tasks.sort(Comparator.naturalOrder());

        listModel.clear();

        for (String t : tasks) {
            listModel.addElement(t);
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(Planirovschik::new);
    }
}
