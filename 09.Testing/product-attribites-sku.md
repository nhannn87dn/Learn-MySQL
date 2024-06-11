# Mô hình quản lý sản phẩm

```sql

-- Tạo bảng Categories
CREATE TABLE Categories (
    id INT AUTO_INCREMENT,
    name VARCHAR(100),
    PRIMARY KEY (id)
);


CREATE TABLE Products (
    id INT AUTO_INCREMENT,
    name VARCHAR(100),
    description TEXT,
    category_id INT
    PRIMARY KEY (id),
    FOREIGN KEY (category_id) REFERENCES Categories(id);
);


CREATE TABLE Attributes (
    id INT AUTO_INCREMENT,
    name VARCHAR(100),
    PRIMARY KEY (id)
);

CREATE TABLE ProductAttributes (
    product_id INT,
    attribute_id INT,
    value VARCHAR(100),
    slug VARCHAR(100),
    PRIMARY KEY (product_id, attribute_id),
    FOREIGN KEY (product_id) REFERENCES Products(id),
    FOREIGN KEY (attribute_id) REFERENCES Attributes(id)
);

CREATE TABLE SKUs (
    id INT AUTO_INCREMENT,
    product_id INT,
    sku VARCHAR(100),
    stock INT,
    price DECIMAL(10, 2),
    PRIMARY KEY (id),
    FOREIGN KEY (product_id) REFERENCES Products(id)
);

CREATE TABLE SKUsAttributes (
    sku_id INT,
    attribute_id INT,
    value VARCHAR(100),
    slug VARCHAR(100),
    PRIMARY KEY (sku_id, attribute_id),
    FOREIGN KEY (sku_id) REFERENCES SKUs(id),
    FOREIGN KEY (attribute_id) REFERENCES Attributes(id)
);

```

Cập nhật dữ liệu


```sql
INSERT INTO Category (name) VALUES ('Áo sơ mi');
-- Adjust to match the structure of the Products table
INSERT INTO Products (name, description, category_id) VALUES ('Áo sơ mi M Trắng', 'Áo sơ mi nam màu trắng, chất liệu cotton', 1);
INSERT INTO Products (name, description, category_id) VALUES ('Áo sơ mi M Xanh', 'Áo sơ mi nam màu xanh, chất liệu cotton', 1);


-- Thêm thuộc tính
INSERT INTO Attributes (name, slug) VALUES ('Size', 'size');
INSERT INTO Attributes (name,slug) VALUES ('Màu', 'mau');

-- Thêm thuộc tính cho sản phẩm
INSERT INTO ProductAttributes (product_id, attribute_id, value, slug) VALUES (1, 1, 'M', 'm');
INSERT INTO ProductAttributes (product_id, attribute_id, value, slug) VALUES (1, 2, 'Trắng', 'trang');
INSERT INTO ProductAttributes (product_id, attribute_id, value, slug) VALUES (1, 2, 'Xanh', 'xanh');
-- Thêm SKU
INSERT INTO SKUs (product_id, sku, stock, price) VALUES (1, 'SM-W-M', 100, 200000);

-- Thêm thuộc tính cho SKU
INSERT INTO SKUsAttributes (sku_id, attribute_id, value, slug) VALUES (1, 1, 'M', 'm');
INSERT INTO SKUsAttributes (sku_id, attribute_id, value, slug) VALUES (1, 2, 'Trắng', 'trang');
INSERT INTO SKUsAttributes (sku_id, attribute_id, value, slug) VALUES (1, 2, 'Xanh', 'xanh');
```


Để lọc sản phẩm trong danh mục ‘Áo sơ mi’ với điều kiện size ‘M’ và màu ‘Trắng’ hoặc ‘Xanh’, bạn có thể sử dụng câu lệnh SQL sau:


```sql
SELECT p.id, p.name
FROM Products p
JOIN Categories c ON p.category_id = c.id
JOIN ProductAttributes pa ON p.id = pa.product_id
JOIN Attributes a ON a.id = pa.attribute_id
WHERE c.id = 1
AND (a.name = 'Size' AND pa.value = 'M')
AND (a.name = 'Màu' AND (pa.value = 'Trắng' OR pa.value = 'Xanh'));

```


Để xử lý các trường hợp khác nhau khi lọc sản phẩm theo `size`, `color`, hoặc cả hai, bạn có thể sử dụng một hàm trong Node.js để xây dựng câu truy vấn SQL dựa trên các tham số đầu vào. Dưới đây là một ví dụ về cách bạn có thể thực hiện điều này:

```js
const express = require('express');
const mysql = require('mysql');

const app = express();
const port = 3000;

// Thiết lập kết nối MySQL
const connection = mysql.createConnection({
  host: 'your_host',
  user: 'your_username',
  password: 'your_password',
  database: 'your_database'
});

connection.connect(err => {
  if (err) throw err;
  console.log('Connected to MySQL Server!');
});

// Route để lấy sản phẩm theo category_id, size và color
app.get('/products', (req, res) => {
  const { category_id, size, color } = req.query;
  let sql = `
    SELECT DISTINCT p.id, p.name
    FROM Products p
    JOIN ProductAttributes pa ON p.id = pa.product_id
    JOIN Attributes a ON a.id = pa.attribute_id
    WHERE p.category_id = ?
  `;

  const params = [category_id];
  const conditions = [];

  if (size) {
    conditions.push("(a.name = 'Size' AND pa.value IN (?))");
    params.push(size.split(','));
  }

  if (color) {
    conditions.push("(a.name = 'Màu' AND pa.value IN (?))");
    params.push(color.split(','));
  }
 
 // Tạo câu lệnh AND giữa các attr khác nhau, và OR khi cùng một thuộc tính mà có nhiều giá trị
  if (conditions.length > 0) {
    sql += " AND (" + conditions.join(" OR ") + ")";
  }

  connection.query(sql, params, (error, results) => {
    if (error) throw error;
    res.json(results);
  });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```


