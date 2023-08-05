# flaskCRUD
示範最簡單的flask資料庫「新增、查詢、刪除、修改」
遵循DevOps的規則，我們將建立一個簡單的Flask應用程式，實現CRUD（創建、讀取、更新和刪除）操作。這個例子使用SQLite作為資料庫，並使用SQLAlchemy來處理資料庫操作。

在開始之前，確保已經安裝了Flask和SQLAlchemy。你可以使用以下指令來安裝：

```
pip install Flask SQLAlchemy
```

接下來，讓我們逐步完成Flask的CRUD範例：

1. **初始化Flask應用程式**：建立`app.py`檔案並初始化Flask應用程式。

```python
# app.py
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
db = SQLAlchemy(app)

# 資料庫模型
class Item(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    description = db.Column(db.Text)

    def __repr__(self):
        return f"<Item {self.name}>"

if __name__ == '__main__':
    app.run(debug=True)
```

2. **建立資料庫表格**：使用Flask的CLI來創建資料庫表格。

在命令列中執行以下指令：

```
flask shell
```

然後在Python REPL中執行以下代碼：

```python
from app import db
db.create_all()
exit()
```

這將創建SQLite資料庫文件`database.db`，並創建`Item`資料表。

3. **實現CRUD路由**：添加用於CRUD操作的路由。

```python
# app.py

# 創建一個新項目
@app.route('/item', methods=['POST'])
def create_item():
    data = request.json
    name = data.get('name')
    description = data.get('description')
    
    if name:
        item = Item(name=name, description=description)
        db.session.add(item)
        db.session.commit()
        return jsonify({'message': 'Item created successfully!', 'item': item.to_dict()}), 201
    else:
        return jsonify({'error': 'Item name is required.'}), 400

# 取得所有項目
@app.route('/item', methods=['GET'])
def get_all_items():
    items = Item.query.all()
    return jsonify({'items': [item.to_dict() for item in items]}), 200

# 取得單一項目
@app.route('/item/<int:item_id>', methods=['GET'])
def get_item(item_id):
    item = Item.query.get(item_id)
    if item:
        return jsonify({'item': item.to_dict()}), 200
    else:
        return jsonify({'error': 'Item not found.'}), 404

# 更新項目
@app.route('/item/<int:item_id>', methods=['PUT'])
def update_item(item_id):
    data = request.json
    name = data.get('name')
    description = data.get('description')

    item = Item.query.get(item_id)
    if item:
        item.name = name if name else item.name
        item.description = description if description else item.description
        db.session.commit()
        return jsonify({'message': 'Item updated successfully!', 'item': item.to_dict()}), 200
    else:
        return jsonify({'error': 'Item not found.'}), 404

# 刪除項目
@app.route('/item/<int:item_id>', methods=['DELETE'])
def delete_item(item_id):
    item = Item.query.get(item_id)
    if item:
        db.session.delete(item)
        db.session.commit()
        return jsonify({'message': 'Item deleted successfully!'}), 200
    else:
        return jsonify({'error': 'Item not found.'}), 404
```

4. **執行應用程式**：在命令列中執行以下指令來運行應用程式：

```
python app.py
```

現在，你的Flask應用程式已經具備了CRUD功能。你可以使用HTTP請求來創建、讀取、更新和刪除項目。以下是一些例子：

- **創建新項目**：使用`POST`請求到`/item`路由，並在請求正文中包含JSON數據。例如：

```json
{
  "name": "Item 1",
  "description": "This is item number 1."
}
```

- **取得所有項目**：使用`GET`請求到`/item`路由。

- **取得單一項目**：使用`GET`請求到`/item/<item_id>`路由，將`<item_id>`替換為項目的ID。

- **更新項目**：使用`PUT`請求到`/item/<item_id>`路由，並在請求正文中包含要更新的JSON數據。

```json
{
  "name": "Updated Item 1",
  "description": "This is the updated item number 1."
}
```

- **刪除項目**：使用`DELETE`請求到`/item/<item_id>`路由，將`<item_id>`替換為要刪除的項目的ID。

請注意，這個範例僅用於演示目的。在實際應用中，你可能需要進一步處理輸入驗證、授權和安全性等問題。同時，使用SQLite作為生產環境的資料庫不是最佳選擇，你可以選擇更適合的資料庫（如PostgreSQL或MySQL）來處理生產環境的數據。
