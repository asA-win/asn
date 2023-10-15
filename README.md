import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class GroceryDataScraper {
  // Method to scrape data from a grocery website and store it in the database
  public static void scrapeAndStoreProductData(String websiteUrl, String cssSelector) {
    String jdbcUrl = "jdbc:mysql://localhost:3306/grocery_comparisonn";
    String username = "root";
    String password = "aswinbros";

    try (Connection connection = DriverManager.getConnection(jdbcUrl, username, password)) {
      Document doc = Jsoup.connect(websiteUrl).get();
      Elements products = doc.select(cssSelector);

      System.out.println("Scraping data from: " + websiteUrl);
      System.out.println("CSS Selector: " + cssSelector);
      System.out.println("Number of products found: " + products.size());

      for (Element product : products) {
        String productName = product.select(".product-item-title").text();
        String price = product.select(".product-price").text();

        // Insert the product data into the database
        boolean success = insertIntoDatabase(connection, productName, Double.parseDouble(price));

        if (success) {
          System.out.println("Inserted product: " + productName);
        } else {
          System.out.println("Failed to insert product: " + productName);
        }
      }
    } catch (SQLException | IOException e) {
      e.printStackTrace();
    }
  }

  // Method to insert data into the MySQL database
  public static boolean insertIntoDatabase(Connection connection, String productName, double productPrice) {
    String insertQuery = "INSERT INTO productss (name, price) VALUES (?, ?)";

    try (PreparedStatement preparedStatement = connection.prepareStatement(insertQuery)) {
      preparedStatement.setString(1, productName);
      preparedStatement.setDouble(2, productPrice);
      int rowsAffected = preparedStatement.executeUpdate();

      if (rowsAffected > 0) {
        return true;
      } else {
        return false;
      }
    } catch (SQLException e) {
      e.printStackTrace();
      return false;
    }
  }

  public static void main(String[] args) {
    GroceryDataScraper.scrapeAndStoreProductData("https://www.walmart.com/browse/food/milk/976759_9176907_4405816", "div.product-item .product-item-title a.product-title-link");
  }
}
