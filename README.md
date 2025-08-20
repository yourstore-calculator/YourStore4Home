# YourStore4Home
<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
  <meta charset="UTF-8" />
  <title>Advanced Price Calculator</title>
  <style>
    body { font-family: 'Segoe UI', sans-serif; background-color: #1e1e1e; color: white; direction: ltr; padding: 30px; }
    select, input, textarea { padding: 8px; margin: 5px 0; width: 100%; border: none; border-radius: 4px; box-sizing: border-box; background-color: #333; color: white; }
    textarea { resize: vertical; min-height: 80px; }
    .section { background-color: #252526; padding: 15px; border-radius: 8px; margin-bottom: 20px; border: 1px solid #333; }
    label { font-weight: bold; color: #00e676; }
    button { background-color: #007acc; color: white; padding: 12px 22px; border: none; margin-top: 10px; border-radius: 5px; cursor: pointer; font-size: 16px; transition: background-color 0.3s; }
    button:hover { background-color: #005a9e; }
    .btn-secondary { background-color: #555; }
    .btn-secondary:hover { background-color: #777; }
    .btn-danger { background-color: #c0392b; }
    .btn-danger:hover { background-color: #a52f22; }
    .btn-primary { background-color: #007acc; }
    .btn-primary:hover { background-color: #005a9e; }
    .results { background: #2b2b2b; padding: 20px; margin-top: 20px; border-radius: 8px; }
    .result-item { background-color: #252526; border: 1px solid #333; border-radius: 8px; padding: 15px; margin-bottom: 15px; }
    .summary { font-weight: bold; color: #00e676; padding-top: 15px; border-top: 2px solid #00e676; margin-top: 15px; }
    .item-header { display: flex; justify-content: space-between; align-items: center; }
    .item-header h4 { margin: 0; font-size: 1.2em; }
    .edit-form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 15px; margin-top: 10px; }
    
    dialog { background-color: #252526; color: white; border: 1px solid #555; border-radius: 8px; width: 90%; max-width: 1000px; padding: 0; }
    dialog::backdrop { background: rgba(0, 0, 0, 0.7); }
    .modal-header { padding: 15px; border-bottom: 1px solid #555; display: flex; justify-content: space-between; align-items: center; }
    .modal-header h2 { margin: 0; }
    .modal-body { padding: 20px; max-height: 70vh; overflow-y: auto; }
    .modal-footer { padding: 15px; border-top: 1px solid #555; text-align: right; }
    .settings-category { background-color: #333; padding: 15px; border-radius: 6px; margin-bottom: 15px; }
    .settings-product { display: grid; grid-template-columns: repeat(auto-fit, minmax(130px, 1fr)); gap: 10px; align-items: center; padding-top: 15px; margin-top: 10px; border-top: 1px solid #444; }
    .product-name { font-weight: bold; font-size: 1.1em; color: #00e676; grid-column: 1 / -1; margin-bottom: 5px; }
  </style>
</head>
<body>

<div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
    <h1 style="margin: 0;">Price Calculator</h1>
    <button onclick="openSettings()" class="btn-secondary">Settings ⚙️</button>
</div>

<div class="section">
    <label for="customerName">Customer Name:</label>
    <input type="text" id="customerName" placeholder="e.g., John Doe" />
    <label for="customerPhone">Customer Phone:</label>
    <input type="text" id="customerPhone" placeholder="e.g., 99455082" />
</div>

<div class="section">
  <label for="mainCategory">Main Category:</label>
  <select id="mainCategory" onchange="loadSubTypes()"></select>
</div>

<div class="section">
  <label for="subType">Sub Type:</label>
  <select id="subType" onchange="setDefaultDimensions()"></select>
</div>

<div class="section">
  <label for="height">Height (meter):</label>
  <input type="number" id="height" step="0.01" value="1" />
  <label for="width">Width (meter):</label>
  <input type="number" id="width" step="0.01" value="1" />
  <label for="quantity">Quantity:</label>
  <input type="number" id="quantity" value="1" />
</div>

<button onclick="addItem()">➕ Add Item</button>
<button onclick="clearAllResults()" class="btn-danger">🗑️ Clear All Results</button>
<button onclick="saveAsWord()" class="btn-secondary">💾 Save as Word</button>

<div class="results" id="results"></div>

<dialog id="settingsModal">
    <div class="modal-header"><h2>Calculator Settings</h2><button onclick="settingsModal.close()" style="background:none;border:none;font-size:1.5em;color:white;cursor:pointer;">&times;</button></div>
    <div class="modal-body" id="settingsContent"></div>
    <div class="modal-footer"><button onclick="addNewCategory()" class="btn-secondary">Add New Category</button><button onclick="resetData()" class="btn-danger">Reset Data to Default</button><button onclick="saveSettings()" class="btn-primary">Save and Close</button></div>
</dialog>

<script>
const SHIPPING_RATE = 48;

const defaultProductData = {
    "Windows": {
        "Window Double Glass Double Frame Fixed": { price: 34, cbm: 0.13, method: 'per_meter' },
        "Window Double Glass Double Frame 1-Way": { price: 34, fixed_component_cost: 39, cbm: 0.13, method: 'per_meter' },
        "Window Double Glass Double Frame 2-Way": { price: 34, fixed_component_cost: 58, cbm: 0.13, method: 'per_meter' },
        "Window Double Glass Single Frame Fixed": { price: 26, cbm: 0.07, method: 'per_meter' },
        "Window Double Glass Single Frame 1-Way": { price: 26, fixed_component_cost: 20, cbm: 0.07, method: 'per_meter' },
        "Window Double Glass Single Frame 2-Way": { price: 26, fixed_component_cost: 32, cbm: 0.07, method: 'per_meter' },
        "Window Single Glass Single Frame Fixed": { price: 20, cbm: 0.07, method: 'per_meter' },
        "Window Single Glass Single Frame 1-Way": { price: 20, fixed_component_cost: 13, cbm: 0.07, method: 'per_meter' },
        "Window Single Glass Single Frame 2-Way": { price: 20, fixed_component_cost: 17, cbm: 0.07, method: 'per_meter' },
        "Sliding Windows": { price: 41, fixed_component_cost: 10, cbm: 0.13, method: 'per_meter' },
        "Electric Windows": { price: 102, cbm: 0.13, method: 'per_meter' },
        "Skylight without Motor": { price: 56, cbm: 0.13, method: 'per_meter' },
        "Skylight with Motor": { price: 145, cbm: 0.13, method: 'per_meter' },
        "Heavy Curtain Wall": { price: 56, cbm: 0.15, method: 'per_meter' },
        "Light Curtain Wall": { price: 45, cbm: 0.15, method: 'per_meter' },
    },
    "Doors": {
        "Entrance Door - Zinc": { price: 66, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "Entrance Door - Stainless Steel": { price: 120, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "Entrance Door - Cast Aluminum": { price: 168, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "WPC Door": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "WPC Door - with Wood": { price: 50, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "WPC Door - with Soundproof Filling": { price: 60, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "WPC Door - with Aluminum Frame": { price: 67, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - with Wood": { price: 75, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Full": { price: 85, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Hidden": { price: 110, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Exterior": { price: 61, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Bathroom Door - New Type": { price: 55, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "Bathroom Door - Old Type": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "Bathroom Door - Hidden Glass": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
    },
    "Sliding Doors": {
        "Interior Sliding Door - Glass": { price: 38, cbm: 0.15, method: 'per_meter' },
        "Interior Sliding Door - Solid": { price: 41, cbm: 0.15, method: 'per_meter' },
        "Exterior Sliding Door - 1 Panel Open": { price: 55, cbm: 0.15, method: 'per_meter' },
        "Exterior Sliding Door - 2 Panels Open": { price: 58, cbm: 0.15, method: 'per_meter' },
        "WPC Sliding Door": { price: 61, cbm: 0.15, method: 'per_meter' },
    },
    "Folding Doors": {
        "Interior Folding Door": { price: 39, cbm: 0.15, method: 'per_meter' },
        "Exterior Folding Door": { price: 56, cbm: 0.15, method: 'per_meter' },
    },
    "Exterior Shutters": {
        "Rolling Shutter": { price: 28, cbm: 0.20, method: 'per_meter' },
    },
    "Garden Gates": {
        "Cast Aluminum Garden Gate": { price: 91, cbm: 0.20, method: 'per_meter' },
    },
    "Barriers": {
        "Stair Barrier": { price: 43, cbm: 0.05, method: 'per_meter' },
        "Bathroom Barrier": { price: 32, cbm: 0.05, method: 'per_meter' },
    }
};
let productData = {};
let resultsList = [];

function saveData() { localStorage.setItem('calculatorProductData', JSON.stringify(productData)); }

function loadData() {
    const savedData = localStorage.getItem('calculatorProductData');
    try {
        const parsedData = JSON.parse(savedData);
        if (savedData && typeof parsedData === 'object' && parsedData !== null && Object.keys(parsedData).length > 0) {
            productData = parsedData;
        } else {
            throw new Error("Invalid or empty data in localStorage");
        }
    } catch (e) {
        productData = JSON.parse(JSON.stringify(defaultProductData));
        saveData();
    }
}

function initializeApp() {
    loadData();
    const mainCat = document.getElementById("mainCategory");
    mainCat.innerHTML = `<option value="">-- Select Category --</option>`;
    Object.keys(productData).forEach(cat => {
        if (typeof productData[cat] === 'object' && productData[cat] !== null) {
            mainCat.innerHTML += `<option value="${cat}">${cat}</option>`;
        }
    });
    loadSubTypes();
}

function loadSubTypes() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subType = document.getElementById("subType");
    subType.innerHTML = "";
    if (mainCatVal && productData[mainCatVal]) {
        Object.keys(productData[mainCatVal]).forEach(sub => {
            subType.innerHTML += `<option value="${sub}">${sub}</option>`;
        });
    }
    setDefaultDimensions();
}

function setDefaultDimensions() {
    const subVal = document.getElementById("subType").value;
    const heightInput = document.getElementById("height");
    const widthInput = document.getElementById("width");
    
    const data = findProductData(subVal);
    if (data && data.method === 'per_unit') {
        heightInput.value = data.std_h || 2.2;
        widthInput.value = data.std_w || 1.0;
    } else {
        heightInput.value = 1;
        widthInput.value = 1;
    }
}

function findProductData(productName) {
    for (const category in productData) {
        if (productData[category][productName]) {
            return productData[category][productName];
        }
    }
    return null;
}

function calculateItemPrice(item) {
    const data = item.data;
    const area = item.h * item.w;
    let basePrice = 0, sizePenalty = 0;

    if (data.method === 'per_unit') {
        basePrice = data.price || 0;
        const stdArea = (data.std_h || 0) * (data.std_w || 0);
        if (stdArea > 0 && area > stdArea) {
            sizePenalty = Math.ceil((area - stdArea) / 0.1) * 2;
        }
    } else {
        basePrice = area * (data.price || 0);
    }
    
    const fixedCost = data.fixed_component_cost || 0;
    const specialCost = data.special === 'add_10' ? 10 : 0;
    const shippingCost = area * (data.cbm || 0) * SHIPPING_RATE;
    const unitPrice = basePrice + sizePenalty + fixedCost + specialCost + shippingCost;
    
    return { unitPrice, totalPrice: unitPrice * item.qty };
}

function addItem() {
    const subVal = document.getElementById("subType").value;
    if (!subVal) { alert("Please select a sub-type."); return; }
    
    const data = findProductData(subVal);
    const newItem = {
        id: Date.now(),
        name: subVal,
        qty: parseInt(document.getElementById("quantity").value) || 1,
        h: parseFloat(document.getElementById("height").value) || 1,
        w: parseFloat(document.getElementById("width").value) || 1,
        itemNotes: "",
        data: JSON.parse(JSON.stringify(data)),
        isEditing: false
    };
    
    const prices = calculateItemPrice(newItem);
    newItem.unitPrice = prices.unitPrice;
    newItem.totalPrice = prices.totalPrice;
    resultsList.push(newItem);
    renderResults();
}

function deleteItem(id) {
    resultsList = resultsList.filter(item => item.id !== id);
    renderResults();
}

function clearAllResults() {
    resultsList = [];
    renderResults();
}

function enterEditMode(id) {
    resultsList.forEach(item => item.isEditing = (item.id === id));
    renderResults();
}

function cancelEdit(id) {
    const item = resultsList.find(i => i.id === id);
    if (item) item.isEditing = false;
    renderResults();
}

function saveItemEdit(id) {
    const item = resultsList.find(i => i.id === id);
    if (item) {
        item.name = document.getElementById(`edit-name-${id}`).value;
        item.qty = parseFloat(document.getElementById(`edit-qty-${id}`).value) || 0;
        item.h = parseFloat(document.getElementById(`edit-h-${id}`).value) || 0;
        item.w = parseFloat(document.getElementById(`edit-w-${id}`).value) || 0;
        item.itemNotes = document.getElementById(`edit-notes-${id}`).value;
        item.unitPrice = parseFloat(document.getElementById(`edit-unitprice-${id}`).value) || 0;
        item.totalPrice = parseFloat(document.getElementById(`edit-totalprice-${id}`).value) || 0;
        item.isEditing = false;
    }
    renderResults();
}

function renderResults() {
    const container = document.getElementById("results");
    container.innerHTML = "";
    let subtotal = 0;

    resultsList.forEach(item => {
        subtotal += item.totalPrice;
        if (item.isEditing) {
            container.innerHTML += `
                <div class="result-item">
                    <div class="edit-form-grid">
                        <div><label>Name</label><input type="text" id="edit-name-${item.id}" value="${item.name}"></div>
                        <div><label>Qty</label><input type="number" id="edit-qty-${item.id}" value="${item.qty}"></div>
                        <div><label>Height</label><input type="number" step="0.01" id="edit-h-${item.id}" value="${item.h}"></div>
                        <div><label>Width</label><input type="number" step="0.01" id="edit-w-${item.id}" value="${item.w}"></div>
                        <div><label>Unit Price</label><input type="number" step="0.01" id="edit-unitprice-${item.id}" value="${item.unitPrice.toFixed(2)}"></div>
                        <div><label>Total Price</label><input type="number" step="0.01" id="edit-totalprice-${item.id}" value="${item.totalPrice.toFixed(2)}"></div>
                    </div>
                    <div style="margin-top: 15px;"><label>Item Notes</label><textarea id="edit-notes-${item.id}">${item.itemNotes}</textarea></div>
                    <div style="text-align: right; margin-top: 10px;">
                        <button onclick="cancelEdit(${item.id})" class="btn-secondary" style="padding: 8px 15px;">Cancel</button>
                        <button onclick="saveItemEdit(${item.id})" class="btn-primary" style="padding: 8px 15px;">Save Changes</button>
                    </div>
                </div>`;
        } else {
            container.innerHTML += `
                <div class="result-item">
                    <div class="item-header">
                        <h4>${item.name}</h4>
                        <div>
                           <button onclick="enterEditMode(${item.id})" class="btn-secondary" style="padding: 6px 12px; margin: 0 5px;">Edit ✏️</button>
                           <button onclick="deleteItem(${item.id})" class="btn-danger" style="padding: 6px 12px; margin: 0;">Delete 🗑️</button>
                        </div>
                    </div>
                    <p><strong>Dimensions:</strong> ${item.h}m x ${item.w}m | <strong>Quantity:</strong> ${item.qty}</p>
                    <p style="font-weight: bold; font-size: 1.1em;">
                        Unit Price: ${item.unitPrice.toFixed(2)} OMR | Total: ${item.totalPrice.toFixed(2)} OMR
                    </p>
                    ${item.itemNotes ? `<p><strong>Notes:</strong> ${item.itemNotes.replace(/\n/g, '<br>')}</p>` : ''}
                </div>`;
        }
    });

    if (resultsList.length > 0) {
        const commission = subtotal * 0.04;
        const total = subtotal + commission;
        container.innerHTML += `
            <div class="summary">
                <div style="display: flex; justify-content: space-between;"><span>Subtotal:</span><span>${subtotal.toFixed(2)} OMR</span></div>
                <div style="display: flex; justify-content: space-between;"><span>Office Commission (4%):</span><span>${commission.toFixed(2)} OMR</span></div>
                <div style="background-color: #00e676; color: #1e1e1e; padding: 15px; border-radius: 5px; text-align: center; margin-top: 10px;">
                    <span style="font-size: 1.3em;">💰 Grand Total</span>
                    <div style="font-size: 2.0em; font-weight: bold;">${total.toFixed(2)} OMR</div>
                </div>
            </div>`;
    }
}

function openSettings() {
    const password = prompt("Enter password to access settings:");
    if (password === "0000") {
        renderSettings();
        document.getElementById('settingsModal').showModal();
    } else if (password) {
        alert("Incorrect password.");
    }
}

function renderSettings() {
    const container = document.getElementById('settingsContent');
    container.innerHTML = '';
    for (const category in productData) {
        let categoryHtml = `<div class="settings-category">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px;">
                <h3>${category}</h3>
                <div>
                    <button onclick="addNewProduct('${category}')" class="btn-secondary" style="margin: 0 5px;">+ Add Product</button>
                    <button onclick="deleteCategory('${category}')" class="btn-danger" style="margin: 0;">Delete Category</button>
                </div>
            </div>`;
        for (const product in productData[category]) {
            const pData = productData[category][product];
            categoryHtml += `<div class="settings-product" data-category="${category}" data-product-name="${product}">
                <div class="product-name">${product}</div>
                <div><label>Method</label><select data-field="method"><option value="per_meter" ${pData.method === 'per_meter' ? 'selected' : ''}>Per Meter</option><option value="per_unit" ${pData.method === 'per_unit' ? 'selected' : ''}>Per Unit</option></select></div>
                <div><label>Price</label><input type="number" step="0.01" value="${pData.price || 0}" data-field="price"></div>
                <div><label>Fixed Cost</label><input type="number" step="0.01" value="${pData.fixed_component_cost || 0}" data-field="fixed_component_cost"></div>
                <div><label>CBM/m²</label><input type="number" step="0.01" value="${pData.cbm || 0}" data-field="cbm"></div>
                <div><label>Std Height</label><input type="number" step="0.01" value="${pData.std_h || 0}" data-field="std_h"></div>
                <div><label>Std Width</label><input type="number" step="0.01" value="${pData.std_w || 0}" data-field="std_w"></div>
                <div><label>Special</label><select data-field="special"><option value="" ${!pData.special ? 'selected' : ''}>None</option><option value="add_10" ${pData.special === 'add_10' ? 'selected' : ''}>Add 10</option></select></div>
                <button onclick="deleteProduct('${category}', '${product}')" class="btn-danger" style="padding: 5px 10px; margin-top: 20px;">Delete</button>
            </div>`;
        }
        categoryHtml += `</div>`;
        container.innerHTML += categoryHtml;
    }
}

function saveSettings() {
    const newProductData = {};
    document.querySelectorAll('.settings-category').forEach(catElement => {
        const categoryName = catElement.querySelector('h3').innerText;
        newProductData[categoryName] = {};
        catElement.querySelectorAll('.settings-product').forEach(pElement => {
            const productName = pElement.dataset.productName;
            const productObject = {};
            pElement.querySelectorAll('input[data-field], select[data-field]').forEach(input => {
                const field = input.dataset.field;
                const value = (input.type === 'number') ? parseFloat(input.value) || 0 : input.value;
                if (value) productObject[field] = value;
            });
            newProductData[categoryName][productName] = productObject;
        });
    });
    productData = newProductData;
    saveData();
    document.getElementById('settingsModal').close();
    initializeApp();
}

function resetData() {
    if (confirm("Are you sure? This will delete all custom data and restore defaults.")) {
        localStorage.removeItem('calculatorProductData');
        loadData();
        renderSettings();
    }
}

function addNewCategory() {
    const newCategoryName = prompt("Enter new category name:");
    if (newCategoryName && !productData[newCategoryName]) {
        productData[newCategoryName] = {};
        renderSettings();
    } else if (newCategoryName) {
        alert("This category already exists.");
    }
}

function deleteCategory(category) {
    if (confirm(`Are you sure you want to delete the entire category "${category}"?`)) {
        delete productData[category];
        renderSettings();
    }
}

function addNewProduct(category) {
    const newProductName = prompt(`Enter new product name for category "${category}":`);
    if (newProductName && !productData[category][newProductName]) {
        productData[category][newProductName] = { method: 'per_meter', price: 0, cbm: 0 };
        renderSettings();
    } else if (newProductName) {
        alert("This product already exists in this category.");
    }
}

function deleteProduct(category, product) {
     if (confirm(`Are you sure you want to delete the product "${product}"?`)) {
        delete productData[category][product];
        renderSettings();
    }
}

function formatNumber(num) {
    return Number(num.toFixed(3).replace(/\.?0+$/, ""));
}

function saveAsWord(){
    if (resultsList.length === 0) { alert("There are no results to save."); return; }
    const customerName = document.getElementById('customerName').value || 'Customer';
    const customerPhone = document.getElementById('customerPhone').value || 'N/A';
    const headerHtml = `<div style="width: 100%; font-family: Arial, sans-serif; text-align: center; margin-bottom: 30px; border-bottom: 2px solid #4472C4; padding-bottom: 20px;"><div style="font-size: 26px; font-weight: bold; color: #4472C4; margin-bottom: 15px;">BLUE WAVES SERVICES LLC</div><table width="100%" style="font-family: Arial, sans-serif; font-size: 15px; color: #4472C4; border-collapse: collapse;"><tr><td style="text-align: left; width: 33%;">OMAN – MUSCAT</td><td style="text-align: center; width: 34%;">SR. NO. : 1595256</td><td style="text-align: right; width: 33%;">TEL: 77 22 45 11 – 90 99 88 10</td></tr></table></div>`;
    const customerHtml = `<div style="background-color: #f2f2f2; border: 1px solid #ddd; color: #333; padding: 14px; font-family: Arial, sans-serif; font-size: 17px; margin-bottom: 25px; border-radius: 5px;"><b>Quotation for: </b> ${customerName} - ${customerPhone}</div>`;
    let tableHtml = `<table border="1" width="100%" style="border-collapse: collapse; font-family: Arial, sans-serif; text-align: center; font-size: 14px;">
                        <thead>
                            <tr style="background-color: #4472C4; color: white;">
                                <th style="padding: 10px;">NO</th>
                                <th style="padding: 10px;">H</th>
                                <th style="padding: 10px;">W</th>
                                <th style="padding: 10px;">m²</th>
                                <th style="padding: 10px;">Q</th>
                                <th style="padding: 10px;">RH/LH</th>
                                <th style="padding: 10px;">STYLE</th>
                                <th style="padding: 10px;">PRICE</th>
                                <th style="padding: 10px;">TOTAL</th>
                                <th style="padding: 10px;">DESCRIPTION</th>
                            </tr>
                        </thead>
                        <tbody>`;
    let subtotal = 0;
    resultsList.forEach((r, index) => {
        subtotal += r.totalPrice;
        const description = `${r.name}${r.itemNotes ? ` - ${r.itemNotes.replace(/\n/g, ', ')}` : ''}`;
        tableHtml += `<tr>
                        <td style="padding: 10px; font-weight: bold;">B${index + 1}</td>
                        <td style="padding: 10px;">${formatNumber(r.h)}</td>
                        <td style="padding: 10px;">${formatNumber(r.w)}</td>
                        <td style="padding: 10px;">${formatNumber(r.h * r.w)}</td>
                        <td style="padding: 10px;">${r.qty}</td>
                        <td style="padding: 10px;"></td>
                        <td style="padding: 10px;"></td>
                        <td style="padding: 10px; background-color: #f2f2f2;">${formatNumber(r.unitPrice)}</td>
                        <td style="padding: 10px; background-color: #f2f2f2; font-weight: bold;">${formatNumber(r.totalPrice)}</td>
                        <td style="padding: 10px; text-align: left; white-space: pre-wrap;">${description}</td>
                      </tr>`;
    });
    tableHtml += `</tbody></table>`;
    const commission = subtotal * 0.04;
    const grandTotal = subtotal + commission;
    const summaryTable = `<table width="45%" align="right" style="border-collapse: collapse; font-family: Arial, sans-serif; margin-top: 25px; font-size: 15px;"><tbody><tr><td style="padding: 14px; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">Subtotal</td><td style="padding: 14px; text-align: right; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(subtotal)} OMR</td></tr><tr><td style="padding: 14px; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">Office Commission (4%)</td><td style="padding: 14px; text-align: right; font-weight: bold; background-color: #f2f2f2; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(commission)} OMR</td></tr><tr style="background-color: #4472C4; color: white;"><td style="padding: 16px; font-weight: bold; border: 1px solid #4472C4; font-size: 1.5em;">Grand Total</td><td style="padding: 16px; text-align: right; font-weight: bold; border: 1px solid #4472C4; font-size: 1.9em; vertical-align: middle;">${formatNumber(grandTotal)} OMR</td></tr></tbody></table>`;
    const footerNotesHtml = `<div style="clear: both; font-family: Arial, sans-serif; margin-top: 50px; text-align: center; border-top: 2px solid #4472C4; padding-top: 15px; font-size: 16px; color: #333;"><p style="margin: 5px 0;"><b>*Price includes delivery*</b></p><p style="margin: 5px 0;"><b>*Price does not include installation*</b></p></div>`;
    const finalHtml = `<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'><head><meta charset='utf-8'><title>Quotation for - ${customerName}</title></head><body dir="ltr" style="padding: 20px;">`+ headerHtml + customerHtml + tableHtml + summaryTable + footerNotesHtml +`</body></html>`;
    const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(finalHtml);
    const fileDownload = document.createElement("a");
    document.body.appendChild(fileDownload);
    fileDownload.href = source;
    fileDownload.download = `Quotation for - ${customerName}.doc`;
    fileDownload.click();
    document.body.removeChild(fileDownload);
}

initializeApp();
</script>

</body>
</html>
