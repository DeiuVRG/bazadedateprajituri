# bazadedateprajituri

import java.sql.*;
import java.util.Scanner;

public class PatiserieApp {

    private static final String URL = "jdbc:mysql://localhost:3306/patiserie";
    private static final String USER = "root";
    private static final String PASSWORD = "root";

    public static void main(String[] args) {
        try (Connection connection = DriverManager.getConnection(URL, USER, PASSWORD)) {
            createTable(connection);
            populateData(connection);

            Scanner scanner = new Scanner(System.in);

            System.out.println("1. Calculează prețul mediu al prăjiturilor");
            System.out.println("2. Caută o prăjitură după denumire");
            System.out.println("3. Numără prăjiturile fabricate într-o anumită lună");
            System.out.print("Alege o opțiune: ");
            int option = scanner.nextInt();
            scanner.nextLine(); // Consumă newline

            switch (option) {
                case 1 -> calculateAveragePrice(connection);
                case 2 -> {
                    System.out.print("Introdu denumirea prăjiturii: ");
                    String name = scanner.nextLine();
                    searchCakeByName(connection, name);
                }
                case 3 -> {
                    System.out.print("Introdu anul (YYYY): ");
                    int year = scanner.nextInt();
                    System.out.print("Introdu luna (MM): ");
                    int month = scanner.nextInt();
                    countCakesByMonth(connection, year, month);
                }
                default -> System.out.println("Opțiune invalidă!");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private static void createTable(Connection connection) throws SQLException {
        String createTableSQL = """
                CREATE TABLE IF NOT EXISTS prajituri (
                    id INT AUTO_INCREMENT PRIMARY KEY,
                    denumire VARCHAR(50),
                    tip VARCHAR(20),
                    ingrediente VARCHAR(255),
                    pret DECIMAL(10, 2),
                    data_fabricatiei DATE
                )
                """;
        try (Statement statement = connection.createStatement()) {
            statement.executeUpdate(createTableSQL);
        }
    }

    private static void populateData(Connection connection) throws SQLException {
        String insertSQL = """
                INSERT INTO prajituri (denumire, tip, ingrediente, pret, data_fabricatiei)
                VALUES (?, ?, ?, ?, ?)
                """;
        try (PreparedStatement ps = connection.prepareStatement(insertSQL)) {
            ps.setString(1, "Tort de ciocolată");
            ps.setString(2, "Tort");
            ps.setString(3, "Ciocolată, făină, ouă, zahăr");
            ps.setBigDecimal(4, new java.math.BigDecimal("45.50"));
            ps.setDate(5, Date.valueOf("2025-01-10"));
            ps.executeUpdate();

            ps.setString(1, "Fursecuri cu stafide");
            ps.setString(2, "Fursecuri");
            ps.setString(3, "Stafide, făină, zahăr, unt");
            ps.setBigDecimal(4, new java.math.BigDecimal("15.75"));
            ps.setDate(5, Date.valueOf("2025-01-05"));
            ps.executeUpdate();
        }
    }

    private static void calculateAveragePrice(Connection connection) throws SQLException {
        String query = "SELECT AVG(pret) AS pret_mediu FROM prajituri";
        try (Statement statement = connection.createStatement();
             ResultSet rs = statement.executeQuery(query)) {
            if (rs.next()) {
                System.out.printf("Prețul mediu al prăjiturilor este: %.2f%n", rs.getDouble("pret_mediu"));
            }
        }
    }

    private static void searchCakeByName(Connection connection, String name) throws SQLException {
        String query = "SELECT * FROM prajituri WHERE denumire = ?";
        try (PreparedStatement ps = connection.prepareStatement(query)) {
            ps.setString(1, name);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    System.out.printf("Denumire: %s, Tip: %s, Ingrediente: %s, Preț: %.2f, Data Fabricării: %s%n",
                            rs.getString("denumire"),
                            rs.getString("tip"),
                            rs.getString("ingrediente"),
                            rs.getDouble("pret"),
                            rs.getDate("data_fabricatiei"));
                } else {
                    System.out.println("Nu există prăjituri cu această denumire.");
                }
            }
        }
    }

    private static void countCakesByMonth(Connection connection, int year, int month) throws SQLException {
        String query = "SELECT COUNT(*) AS total FROM prajituri WHERE YEAR(data_fabricatiei) = ? AND MONTH(data_fabricatiei) = ?";
        try (PreparedStatement ps = connection.prepareStatement(query)) {
            ps.setInt(1, year);
            ps.setInt(2, month);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    System.out.printf("Numărul de prăjituri fabricate în %d-%02d este: %d%n", year, month, rs.getInt("total"));
                }
            }
        }
    }
}

