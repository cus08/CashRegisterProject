import java.io.*;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.regex.*;

class User {
    String username;
    String password;

    User(String username, String password) {
        this.username = username;
        this.password = password;
    }
}

class Order {
    String itemName;
    int quantity;
    double price;

    Order(String itemName, int quantity, double price) {
        this.itemName = itemName;
        this.quantity = quantity;
        this.price = price;
    }

    public double getTotal() {
        return quantity * price;
    }

    public String toString() {
        return itemName + " - Qty: " + quantity + ", Price: ₱" + price + ", Total: ₱" + getTotal();
    }
}

public class Main {
    static Scanner scanner = new Scanner(System.in);
    static Map<String, User> users = new HashMap<>();
    static ArrayList<Order> orders = new ArrayList<>();
    static String loggedInUser = null;

    public static void main(String[] args) {
        while (true) {
            System.out.println("\nWelcome to Cash Register System");
            System.out.println("1. Sign Up");
            System.out.println("2. Login");
            System.out.println("3. Exit");

            System.out.print("Choose: ");
            String choice = scanner.nextLine();

            switch (choice) {
                case "1":
                    signUp();
                    break;
                case "2":
                    login();
                    break;
                case "3":
                    System.out.println("Thank you for using the system.");
                    return;
                default:
                    System.out.println("Invalid choice.");
            }
        }
    }

    static void signUp() {
        System.out.print("Enter username: ");
        String username = scanner.nextLine();

        if (users.containsKey(username)) {
            System.out.println("Username already exists.");
            return;
        }

        System.out.print("Enter password (at least 6 chars, 1 uppercase, 1 digit): ");
        String password = scanner.nextLine();

        if (!isValidPassword(password)) {
            System.out.println("Invalid password format.");
            return;
        }

        users.put(username, new User(username, password));
        System.out.println("Sign up successful.");
    }

    static boolean isValidPassword(String password) {
        Pattern pattern = Pattern.compile("^(?=.*[A-Z])(?=.*\\d).{6,}$");
        return pattern.matcher(password).matches();
    }

    static void login() {
        System.out.print("Enter username: ");
        String username = scanner.nextLine();

        System.out.print("Enter password: ");
        String password = scanner.nextLine();

        User user = users.get(username);

        if (user != null && user.password.equals(password)) {
            loggedInUser = username;
            System.out.println("Login successful. Welcome, " + username + "!");
            manageOrders();
        } else {
            System.out.println("Invalid credentials.");
        }
    }

    static void manageOrders() {
        orders.clear();

        while (true) {
            System.out.println("\n1. Add Order");
            System.out.println("2. Update Quantity");
            System.out.println("3. Remove Order");
            System.out.println("4. Display Orders");
            System.out.println("5. Checkout");
            System.out.println("6. Logout");

            System.out.print("Choose: ");
            String choice = scanner.nextLine();

            switch (choice) {
                case "1":
                    addOrder();
                    break;
                case "2":
                    updateQuantity();
                    break;
                case "3":
                    removeOrder();
                    break;
                case "4":
                    displayOrders();
                    break;
                case "5":
                    checkout();
                    return;
                case "6":
                    System.out.println("Logged out.");
                    return;
                default:
                    System.out.println("Invalid choice.");
            }
        }
    }

    static void addOrder() {
        try {
            System.out.print("Enter item name: ");
            String name = scanner.nextLine();

            System.out.print("Enter quantity: ");
            int qty = Integer.parseInt(scanner.nextLine());

            System.out.print("Enter price: ");
            double price = Double.parseDouble(scanner.nextLine());

            orders.add(new Order(name, qty, price));
            System.out.println("Order added.");
        } catch (Exception e) {
            System.out.println("Invalid input. Try again.");
        }
    }

    static void updateQuantity() {
        displayOrders();
        try {
            System.out.print("Enter order number to update: ");
            int index = Integer.parseInt(scanner.nextLine()) - 1;

            if (index >= 0 && index < orders.size()) {
                System.out.print("Enter new quantity: ");
                int newQty = Integer.parseInt(scanner.nextLine());
                orders.get(index).quantity = newQty;
                System.out.println("Quantity updated.");
            } else {
                System.out.println("Invalid order number.");
            }
        } catch (Exception e) {
            System.out.println("Error. Try again.");
        }
    }

    static void removeOrder() {
        displayOrders();
        try {
            System.out.print("Enter order number to remove: ");
            int index = Integer.parseInt(scanner.nextLine()) - 1;

            if (index >= 0 && index < orders.size()) {
                orders.remove(index);
                System.out.println("Order removed.");
            } else {
                System.out.println("Invalid order number.");
            }
        } catch (Exception e) {
            System.out.println("Error. Try again.");
        }
    }

    static void displayOrders() {
        if (orders.isEmpty()) {
            System.out.println("No orders.");
            return;
        }

        System.out.println("\n--- Orders ---");
        int i = 1;
        double total = 0;
        for (Order order : orders) {
            System.out.println((i++) + ". " + order);
            total += order.getTotal();
        }
        System.out.println("Total: ₱" + total);
    }

    static void checkout() {
        if (orders.isEmpty()) {
            System.out.println("No items to checkout.");
            return;
        }

        double total = 0;
        StringBuilder receipt = new StringBuilder();

        receipt.append("=====================================\n");
        receipt.append("Date/Time: ").append(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date())).append("\n");
        receipt.append("Cashier: ").append(loggedInUser).append("\n");
        receipt.append("Items:\n");

        for (Order order : orders) {
            receipt.append("- ").append(order.itemName)
                    .append(", Qty: ").append(order.quantity)
                    .append(", Price: ₱").append(order.price)
                    .append(", Total: ₱").append(order.getTotal()).append("\n");
            total += order.getTotal();
        }

        receipt.append("Total Amount: ₱").append(total).append("\n");
        receipt.append("=====================================\n\n");

        try (FileWriter writer = new FileWriter("transactions.txt", true)) {
            writer.write(receipt.toString());
            System.out.println("\nCheckout complete! Transaction saved.");
        } catch (IOException e) {
            System.out.println("Failed to write transaction.");
        }

        orders.clear();
    }
}
