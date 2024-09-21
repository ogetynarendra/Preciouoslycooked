<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Previously Cooked</title>
    <style>
        body { font-family: Arial, sans-serif; }
        .container { width: 50%; margin: 0 auto; padding: 20px; }
        input, button { padding: 10px; margin: 5px; display: block; }
        table { width: 100%; margin-top: 20px; }
        th, td { padding: 10px; text-align: left; }
    </style>
</head>
<body>

<div class="container">
    <h1>Menu for <span id="day"></span> - <span id="date"></span></h1>
    
    <table border="1">
        <thead>
            <tr>
                <th>Item</th>
                <th>Price ($)</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody id="menuItems">
            <!-- Menu items will be displayed here -->
        </tbody>
    </table>

    <h3>Add a New Menu Item</h3>
    <form id="addItemForm">
        <input type="text" id="newItemName" placeholder="Item Name" required>
        <input type="number" id="newItemPrice" placeholder="Item Price ($)" required>
        <button type="submit">Add Item</button>
    </form>

    <h2>Interested? Order Now!</h2>
    <form id="orderForm">
        <input type="text" id="name" placeholder="Your Name" required>
        <input type="number" id="quantity" placeholder="Quantity" required>
        <input type="text" id="address" placeholder="Your Address" required>
        <input type="tel" id="contactNumber" placeholder="Contact Number" required>
        <button type="submit">Submit Order</button>
    </form>
</div>

<!-- Firebase SDKs -->
<script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.0.0/firebase-firestore.js"></script>

<script>
    // Firebase Config (Add your Firebase credentials here)
    const firebaseConfig = {
        apiKey: "YOUR_API_KEY",
        authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
        projectId: "YOUR_PROJECT_ID",
        storageBucket: "YOUR_PROJECT_ID.appspot.com",
        messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
        appId: "YOUR_APP_ID"
    };

    // Initialize Firebase
    const app = firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();

    const dayElement = document.getElementById('day');
    const dateElement = document.getElementById('date');
    const menuItemsElement = document.getElementById('menuItems');
    const addItemForm = document.getElementById('addItemForm');
    const orderForm = document.getElementById('orderForm');

    // Fetch the menu for the day
    async function fetchMenu() {
        const menuDoc = await db.collection('menus').doc('monday').get();
        const menuData = menuDoc.data();

        // Display day and date
        dayElement.textContent = menuData.day;
        dateElement.textContent = menuData.date;

        // Display menu items
        menuItemsElement.innerHTML = '';
        menuData.items.forEach((item, index) => {
            menuItemsElement.innerHTML += `
                <tr>
                    <td>${item.name}</td>
                    <td>${item.price}</td>
                    <td><button onclick="deleteItem(${index})">Delete</button></td>
                </tr>
            `;
        });
    }

    // Add a new item to the menu
    addItemForm.addEventListener('submit', async (e) => {
        e.preventDefault();
        const newItemName = document.getElementById('newItemName').value;
        const newItemPrice = document.getElementById('newItemPrice').value;

        const menuDoc = await db.collection('menus').doc('monday').get();
        const menuData = menuDoc.data();

        // Add new item
        menuData.items.push({ name: newItemName, price: newItemPrice });
        await db.collection('menus').doc('monday').set(menuData);

        fetchMenu();  // Refresh the menu
        addItemForm.reset();
    });

    // Delete an item from the menu
    async function deleteItem(index) {
        const menuDoc = await db.collection('menus').doc('monday').get();
        const menuData = menuDoc.data();

        // Remove item by index
        menuData.items.splice(index, 1);
        await db.collection('menus').doc('monday').set(menuData);

        fetchMenu();  // Refresh the menu
    }

    // Submit order form
    orderForm.addEventListener('submit', async (e) => {
        e.preventDefault();
        const name = document.getElementById('name').value;
        const quantity = document.getElementById('quantity').value;
        const address = document.getElementById('address').value;
        const contactNumber = document.getElementById('contactNumber').value;

        // Save order details
        await db.collection('orders').add({
            name: name,
            quantity: quantity,
            address: address,
            contactNumber: contactNumber
        });

        alert('Order Submitted!');
        orderForm.reset();
    });

    // Fetch the menu when the page loads
    fetchMenu();
</script>

</body>
</html>
