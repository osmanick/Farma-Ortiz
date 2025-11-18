<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App de Ventas e Inventario</title>
    <!-- Carga Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Configuración de Tailwind para usar la fuente Inter -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                },
            }
        }
    </script>
    
    <style>
        /* Estilo para el cargador inicial */
        #loader {
            display: flex;
            position: fixed;
            inset: 0;
            background: white;
            align-items: center;
            justify-content: center;
            z-index: 999;
            transition: opacity 0.3s ease;
        }
        .spinner {
            border: 4px solid rgba(0,0,0,0.1);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            border-left-color: #0ea5e9; /* Azul de Tailwind */
            animation: spin 1s ease infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gray-100 font-sans">

    <!-- Cargador inicial -->
    <div id="loader">
        <div class="spinner"></div>
    </div>

    <!-- Contenedor principal de la App -->
    <div id="app-container" class="max-w-6xl mx-auto p-4 sm:p-6 lg:p-8 opacity-0 transition-opacity duration-300">
        
        <header class="mb-8">
            <h1 class="text-3xl font-bold text-gray-900">App de Ventas e Inventario</h1>
            <p class="text-sm text-gray-500">Tu ID de usuario es: <code id="user-id-display" class="font-mono bg-gray-200 px-1 rounded">Cargando...</code></p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            
            <!-- Columna del Formulario -->
            <div class="lg:col-span-1 bg-white p-6 rounded-lg shadow-md h-fit">
                <h2 id="form-title" class="text-xl font-semibold mb-4">Agregar Nuevo Producto</h2>
                <form id="product-form" class="space-y-4">
                    <!-- Campo oculto para el ID del producto durante la edición -->
                    <input type="hidden" id="product-id">

                    <div>
                        <label for="product-name" class="block text-sm font-medium text-gray-700">Nombre del Producto</label>
                        <input type="text" id="product-name" name="product-name" required class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>

                    <div>
                        <label for="product-price" class="block text-sm font-medium text-gray-700">Precio (USD)</label>
                        <input type="number" id="product-price" name="product-price" min="0" step="0.01" required class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>

                    <div>
                        <label for="product-quantity" class="block text-sm font-medium text-gray-700">Cantidad en Stock</label>
                        <input type="number" id="product-quantity" name="product-quantity" min="0" step="1" required class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500">
                    </div>

                    <!-- Botones de acción del formulario -->
                    <div id="add-button-container">
                        <button type="submit" class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition-all">
                            Agregar Producto
                        </button>
                    </div>

                    <div id="edit-buttons-container" class="hidden space-y-2">
                        <button type="submit" class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 transition-all">
                            Actualizar Producto
                        </button>
                        <button type="button" id="cancel-edit-button" class="w-full flex justify-center py-2 px-4 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-all">
                            Cancelar Edición
                        </button>
                    </div>
                </form>
            </div>

            <!-- Columna de la Lista de Inventario -->
            <div class="lg:col-span-2 bg-white p-6 rounded-lg shadow-md">
                <h2 class="text-xl font-semibold mb-4">Inventario Actual</h2>
                
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Producto</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Precio</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Stock</th>
                                <th scope="col" class="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">Acciones</th>
                            </tr>
                        </thead>
                        <tbody id="inventory-body" class="bg-white divide-y divide-gray-200">
                            <!-- Filas de productos se insertarán aquí dinámicamente -->
                            <tr id="no-products-row">
                                <td colspan="4" class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">
                                    No hay productos en el inventario.
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <!-- Modal de Alerta Personalizado -->
    <div id="alert-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden transition-opacity" onclick="this.classList.add('hidden')">
        <div class="bg-white p-6 rounded-lg shadow-xl max-w-sm mx-auto" onclick="event.stopPropagation()">
            <h3 id="alert-title" class="text-lg font-medium text-gray-900">Notificación</h3>
            <p id="alert-message" class="mt-2 text-sm text-gray-600">Mensaje de alerta.</p>
            <div class="mt-4">
                <button type="button" id="alert-close-button" class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-blue-600 text-base font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 sm:text-sm">
                    Cerrar
                </button>
            </div>
        </div>
    </div>

    <!-- Scripts de Firebase -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { 
            getAuth, 
            signInAnonymously, 
            signInWithCustomToken, 
            onAuthStateChanged 
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { 
            getFirestore, 
            doc, 
            collection, 
            addDoc, 
            setDoc, 
            updateDoc, 
            deleteDoc, 
            onSnapshot,
            query,
            setLogLevel
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- VARIABLES GLOBALES ---
        let db, auth, app, userId, inventoryCollectionRef;
        const productsCache = new Map(); // Caché para búsquedas rápidas

        // --- ELEMENTOS DEL DOM ---
        const loader = document.getElementById('loader');
        const appContainer = document.getElementById('app-container');
        const productForm = document.getElementById('product-form');
        const formTitle = document.getElementById('form-title');
        const productNameInput = document.getElementById('product-name');
        const productPriceInput = document.getElementById('product-price');
        const productQuantityInput = document.getElementById('product-quantity');
        const productIdInput = document.getElementById('product-id');
        const addButtonContainer = document.getElementById('add-button-container');
        const editButtonsContainer = document.getElementById('edit-buttons-container');
        const cancelEditButton = document.getElementById('cancel-edit-button');
        const inventoryBody = document.getElementById('inventory-body');
        const noProductsRow = document.getElementById('no-products-row');
        const userIdDisplay = document.getElementById('user-id-display');

        // Modal de Alerta
        const alertModal = document.getElementById('alert-modal');
        const alertTitle = document.getElementById('alert-title');
        const alertMessage = document.getElementById('alert-message');
        const alertCloseButton = document.getElementById('alert-close-button');

        // --- FUNCIONES DE LA APLICACIÓN ---

        /**
         * Muestra el modal de alerta personalizado.
         * @param {string} message - El mensaje a mostrar.
         * @param {string} [title='Notificación'] - El título del modal.
         */
        function showAlert(message, title = 'Notificación') {
            alertTitle.textContent = title;
            alertMessage.textContent = message;
            alertModal.classList.remove('hidden');
        }

        // Cierra el modal de alerta
        alertCloseButton.onclick = () => alertModal.classList.add('hidden');

        /**
         * Resetea el formulario al estado inicial (modo "Agregar").
         */
        function resetForm() {
            productForm.reset();
            productIdInput.value = '';
            formTitle.textContent = 'Agregar Nuevo Producto';
            editButtonsContainer.classList.add('hidden');
            addButtonContainer.classList.remove('hidden');
        }

        /**
         * Prepara el formulario para editar un producto existente.
         * @param {string} docId - El ID del documento del producto.
         */
        function setupEditForm(docId) {
            const product = productsCache.get(docId);
            if (!product) return;

            productNameInput.value = product.name;
            productPriceInput.value = product.price;
            productQuantityInput.value = product.quantity;
            productIdInput.value = docId;

            formTitle.textContent = 'Editar Producto';
            addButtonContainer.classList.add('hidden');
            editButtonsContainer.classList.remove('hidden');

            // Scroll al formulario para visibilidad en móviles
            productForm.scrollIntoView({ behavior: 'smooth' });
        }

        /**
         * Renderiza una fila de producto en la tabla de inventario.
         * Actualiza o crea la fila según sea necesario.
         * @param {object} productData - Los datos del producto.
         * @param {string} docId - El ID del documento del producto.
         */
        function renderProductRow(productData, docId) {
            // Ocultar la fila "sin productos" si existe
            if (noProductsRow) {
                noProductsRow.style.display = 'none';
            }

            const { name, price, quantity } = productData;
            
            // Buscar si la fila ya existe
            let row = document.getElementById(docId);

            // Si no existe, crear una nueva
            if (!row) {
                row = document.createElement('tr');
                row.id = docId;
                inventoryBody.appendChild(row);
            }
            
            // Formatear precio
            const formattedPrice = typeof price === 'number' ? price.toFixed(2) : 'N/A';

            // Actualizar el contenido de la fila
            row.innerHTML = `
                <td class="px-6 py-4 whitespace-nowrap">
                    <div class="text-sm font-medium text-gray-900">${name}</div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                    <div class="text-sm text-gray-900">$${formattedPrice}</div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                    <span class="text-sm ${quantity <= 5 ? 'text-red-600 font-bold' : 'text-gray-900'}">
                        ${quantity}
                    </span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium space-x-2">
                    <button data-id="${docId}" class="sell-button text-white bg-yellow-500 hover:bg-yellow-600 px-3 py-1 rounded-md shadow-sm transition-all text-xs">Vender 1</button>
                    <button data-id="${docId}" class="edit-button text-white bg-blue-500 hover:bg-blue-600 px-3 py-1 rounded-md shadow-sm transition-all text-xs">Editar</button>
                    <button data-id="${docId}" class="delete-button text-white bg-red-600 hover:bg-red-700 px-3 py-1 rounded-md shadow-sm transition-all text-xs">Eliminar</button>
                </td>
            `;
        }

        /**
         * Maneja el envío del formulario (tanto para agregar como para actualizar).
         * @param {Event} e - El evento de envío del formulario.
         */
        async function handleFormSubmit(e) {
            e.preventDefault();
            const docId = productIdInput.value;
            const isEditing = !!docId; // Estamos editando si hay un ID

            const productData = {
                name: productNameInput.value,
                price: parseFloat(productPriceInput.value),
                quantity: parseInt(productQuantityInput.value, 10)
            };

            // Validaciones simples
            if (!productData.name || isNaN(productData.price) || isNaN(productData.quantity)) {
                showAlert('Por favor, completa todos los campos con valores válidos.', 'Error de Validación');
                return;
            }

            try {
                if (isEditing) {
                    // Actualizar documento existente
                    const docRef = doc(db, inventoryCollectionRef.path, docId);
                    await updateDoc(docRef, productData);
                    showAlert('Producto actualizado con éxito.');
                } else {
                    // Agregar nuevo documento
                    await addDoc(inventoryCollectionRef, productData);
                    showAlert('Producto agregado con éxito.');
                }
                resetForm(); // Limpiar formulario después de la operación
            } catch (error) {
                console.error("Error al guardar el producto: ", error);
                showAlert(`Error al guardar el producto: ${error.message}`, 'Error de Base de Datos');
            }
        }

        /**
         * Maneja los clics en los botones de la tabla de inventario (Vender, Editar, Eliminar).
         * @param {Event} e - El evento de clic.
         */
        async function handleTableClick(e) {
            const target = e.target;
            const docId = target.dataset.id;
            
            if (!docId) return; // No se hizo clic en un botón con ID

            const product = productsCache.get(docId);
            if (!product) return;

            try {
                // --- Acción: Vender ---
                if (target.classList.contains('sell-button')) {
                    if (product.quantity > 0) {
                        const newQuantity = product.quantity - 1;
                        const docRef = doc(db, inventoryCollectionRef.path, docId);
                        await updateDoc(docRef, { quantity: newQuantity });
                        showAlert(`Venta registrada. Quedan ${newQuantity} unidades de "${product.name}".`);
                    } else {
                        showAlert(`No hay stock de "${product.name}" para vender.`, 'Stock Insuficiente');
                    }
                }
                
                // --- Acción: Editar ---
                else if (target.classList.contains('edit-button')) {
                    setupEditForm(docId);
                }
                
                // --- Acción: Eliminar ---
                else if (target.classList.contains('delete-button')) {
                    // Usamos el modal para confirmar (mejor que window.confirm)
                    // (Para este ejemplo, eliminamos directamente, pero un modal de confirmación sería ideal aquí)
                    const docRef = doc(db, inventoryCollectionRef.path, docId);
                    await deleteDoc(docRef);
                    showAlert(`Producto "${product.name}" eliminado.`);
                }
            } catch (error) {
                console.error("Error en la acción del producto: ", error);
                showAlert(`Error: ${error.message}`, 'Error de Operación');
            }
        }

        /**
         * Configura el listener de Firestore para sincronizar el inventario en tiempo real.
         */
        function setupInventoryListener() {
            if (!inventoryCollectionRef) return;

            // Usamos un query para (opcionalmente) ordenar, aunque Firestore no garantiza orden sin índices
            const q = query(inventoryCollectionRef);

            onSnapshot(q, (snapshot) => {
                if (snapshot.empty) {
                    inventoryBody.innerHTML = ''; // Limpiar el cuerpo
                    inventoryBody.appendChild(noProductsRow);
                    noProductsRow.style.display = 'table-row';
                    productsCache.clear();
                    return;
                }

                snapshot.docChanges().forEach((change) => {
                    const docData = change.doc.data();
                    const docId = change.doc.id;

                    if (change.type === "added" || change.type === "modified") {
                        // Actualizar caché y renderizar fila
                        productsCache.set(docId, docData);
                        renderProductRow(docData, docId);
                    } 
                    
                    if (change.type === "removed") {
                        // Eliminar de caché y del DOM
                        productsCache.delete(docId);
                        const row = document.getElementById(docId);
                        if (row) {
                            row.remove();
                        }
                    }
                });

                // Comprobar si la tabla está vacía después de los cambios
                if (productsCache.size === 0) {
                    inventoryBody.innerHTML = '';
                    inventoryBody.appendChild(noProductsRow);
                    noProductsRow.style.display = 'table-row';
                }

            }, (error) => {
                console.error("Error al escuchar el inventario: ", error);
                showAlert('Error al conectar con la base de datos. Intenta recargar la página.', 'Error de Conexión');
            });
        }

        /**
         * Función principal de inicialización y autenticación.
         */
        async function main() {
            // --- 1. Configuración de Firebase ---
            // Asegurarse de que las variables globales están definidas
            const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
            let firebaseConfig;
            try {
                firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
                if (!firebaseConfig.apiKey) {
                    throw new Error("Configuración de Firebase no válida.");
                }
            } catch (e) {
                console.error("Error al parsear __firebase_config:", e);
                showAlert('Error crítico de configuración. La aplicación no puede iniciarse.', 'Error de Configuración');
                loader.style.opacity = '0';
                loader.style.display = 'none'; // Ocultar loader en error
                return;
            }

            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);
            setLogLevel('debug'); // Habilitar logs de Firestore

            // --- 2. Autenticación ---
            onAuthStateChanged(auth, (user) => {
                if (user) {
                    userId = user.uid;
                    userIdDisplay.textContent = userId; // Mostrar ID de usuario

                    // --- 3. Definir la ruta de la colección ---
                    // Almacenamos el inventario en una colección PÚBLICA (compartida)
                    inventoryCollectionRef = collection(db, 'artifacts', appId, 'public', 'data', 'inventory');
                    
                    // --- 4. Configurar el listener de Firestore ---
                    // Esto solo se ejecuta DESPUÉS de que la autenticación y la configuración de la ruta estén listas
                    setupInventoryListener();

                    // --- 5. Mostrar la aplicación ---
                    loader.style.opacity = '0';
                    setTimeout(() => {
                        loader.style.display = 'none';
                        appContainer.style.opacity = '1';
                    }, 300); // Coincidir con la transición
                }
                // Si no hay usuario, el proceso de inicio de sesión se encargará de ello
            });

            // Iniciar sesión
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
                // onAuthStateChanged se activará y ejecutará el resto de la lógica
            } catch (error) {
                console.error("Error de autenticación: ", error);
                showAlert('No se pudo autenticar. La aplicación no funcionará correctamente.', 'Error de Autenticación');
                loader.style.opacity = '0';
                loader.style.display = 'none';
            }
        }

        // --- INICIAR LA APLICACIÓN ---
        
        // Registrar listeners de eventos del DOM
        productForm.addEventListener('submit', handleFormSubmit);
        cancelEditButton.addEventListener('click', resetForm);
        inventoryBody.addEventListener('click', handleTableClick);

        // Iniciar la lógica principal cuando el DOM esté listo
        document.addEventListener('DOMContentLoaded', main);

    </script>
</body>
</html>
