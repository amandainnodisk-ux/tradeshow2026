<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tradeshow Budget Allocator</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Use Inter font family -->
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7fafc; /* Light gray background */
        }
        .container {
            max-width: 900px;
        }
        .budget-card {
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="loading-overlay" class="fixed inset-0 bg-gray-100 bg-opacity-75 flex items-center justify-center z-50">
        <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-indigo-600"></div>
        <p class="ml-3 text-indigo-700">Loading data...</p>
    </div>

    <div class="container mx-auto">
        <h1 class="text-3xl md:text-4xl font-extrabold text-indigo-800 mb-6 text-center">Tradeshow Budget Allocator</h1>

        <!-- Budget Summary Card -->
        <div class="budget-card bg-white p-6 md:p-8 rounded-xl mb-8">
            <h2 class="text-xl font-semibold text-gray-700 mb-4 border-b pb-2">Global Budget Summary</h2>

            <div class="flex flex-col sm:flex-row justify-between items-center space-y-4 sm:space-y-0 sm:space-x-4 mb-6">
                <div class="w-full sm:w-1/3">
                    <label for="totalBudgetInput" class="block text-sm font-medium text-gray-600 mb-1">Total Global Budget Limit ($)</label>
                    <input type="number" id="totalBudgetInput" value="100000" min="0"
                           class="w-full p-2 border border-indigo-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 text-lg font-bold">
                </div>

                <div class="w-full sm:w-1/3 bg-green-50 p-4 rounded-lg border-2 border-green-200">
                    <p class="text-sm font-medium text-green-700">Total Allocated (Active Shows)</p>
                    <p id="allocatedBudget" class="text-2xl font-extrabold text-green-800">$0.00</p>
                </div>

                <div class="w-full sm:w-1/3 bg-yellow-50 p-4 rounded-lg border-2 border-yellow-200">
                    <p class="text-sm font-medium text-yellow-700">Remaining Global Budget</p>
                    <p id="remainingBudget" class="text-2xl font-extrabold text-yellow-800">$100,000.00</p>
                </div>
            </div>

            <!-- Progress Bar -->
            <div class="w-full bg-gray-200 rounded-full h-3">
                <div id="budgetProgressBar" class="bg-indigo-600 h-3 rounded-full transition-all duration-500" style="width: 0%;"></div>
            </div>
            <p id="statusMessage" class="mt-2 text-sm text-center font-medium text-indigo-600"></p>
        </div>

        <!-- Tradeshow Selection and Item Management -->
        <div class="budget-card bg-white p-6 md:p-8 rounded-xl">
            
            <!-- Global / Active Shows Checkboxes -->
            <div class="mb-6 pb-4 border-b">
                <h2 class="text-xl font-semibold text-gray-700 mb-2">1. Select Active Tradeshows</h2>
                <p class="text-sm text-gray-500 mb-3">Check the shows whose budgets should be included in the Global Allocated Total.</p>
                <div id="activeShowCheckboxes" class="flex flex-wrap gap-2 p-2 bg-gray-100 rounded-lg border border-gray-300">
                    <!-- Checkboxes rendered here -->
                </div>
            </div>
            
            <!-- Item Editing Selector -->
            <div class="mb-6 pb-4 border-b">
                <h2 class="text-xl font-semibold text-gray-700 mb-2">2. Manage Items for a Specific Show</h2>
                <label for="editingShowSelector" class="block text-sm font-medium text-gray-600 mb-2">Select Show for Item Management:</label>
                <select id="editingShowSelector" class="w-full p-2 border border-indigo-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 text-base"></select>
            </div>
            
            <!-- Show-Specific Summary -->
            <div id="showSummary" class="bg-indigo-50 p-4 rounded-lg border border-indigo-200 text-sm mb-6">
                <p id="showAllocated" class="font-semibold text-indigo-800">Please select a show above to manage its budget items.</p>
            </div>

            <!-- Item Selection Panel (Conditional visibility) -->
            <div id="itemManagementPanel" class="hidden">
                <h3 class="text-lg font-bold text-gray-700 mb-4">Budget Items for <span id="currentShowName" class="text-indigo-600"></span></h3>

                <!-- A. BOOTH SPACE Selection -->
                <div class="mb-6 p-4 border rounded-lg bg-gray-50">
                    <h4 class="text-base font-semibold text-gray-700 mb-3 flex justify-between items-center">
                        Booth Space (Select ONE)
                        <span id="boothSpaceCost" class="font-extrabold text-indigo-700"></span>
                    </h4>
                    <div id="boothSpaceOptions" class="flex flex-wrap gap-4">
                        <!-- Radio buttons rendered here -->
                    </div>
                </div>

                <!-- B. BOOTH BUILDING COSTS Selection -->
                <div class="mb-6 p-4 border rounded-lg bg-gray-50">
                    <h4 class="text-base font-semibold text-gray-700 mb-3 flex justify-between items-center">
                        Booth Building Costs (Select ONE)
                        <span id="buildingCostCost" class="font-extrabold text-indigo-700"></span>
                    </h4>
                    <div id="buildingCostOptions" class="flex flex-wrap gap-4">
                        <!-- Radio buttons rendered here -->
                    </div>
                </div>

                <!-- C. CUSTOM ITEMS List and Form -->
                <div class="mb-6 p-4 border rounded-lg bg-white">
                    <h4 class="text-base font-semibold text-gray-700 mb-3">Other Custom Expenses</h4>
                    
                    <!-- Item List Header -->
                    <div class="hidden sm:grid grid-cols-12 gap-4 text-xs font-bold text-gray-500 border-b pb-1 mb-2">
                        <div class="col-span-6">Description</div>
                        <div class="col-span-3 text-right">Cost ($)</div>
                        <div class="col-span-3"></div>
                    </div>
                    
                    <!-- Item List -->
                    <div id="customItemsList" class="space-y-2">
                        <!-- Custom items injected here -->
                        <p id="noCustomItemsMessage" class="text-center text-gray-500 italic text-sm">No custom items added yet.</p>
                    </div>

                    <!-- Add New Custom Item Form -->
                    <div class="mt-4 pt-4 border-t border-dashed border-gray-300 flex flex-col md:flex-row space-y-2 md:space-y-0 md:space-x-2 items-end">
                        <div class="w-full md:w-3/5">
                            <input type="text" id="newItemName" placeholder="e.g., Lead Scanner Rental"
                                class="w-full p-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 text-sm">
                        </div>
                        <div class="w-full md:w-1/5">
                            <input type="number" id="newItemCost" placeholder="1500" min="0"
                                class="w-full p-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 text-sm text-right">
                        </div>
                        <button id="addItemBtn" class="w-full md:w-1/5 bg-indigo-500 hover:bg-indigo-600 text-white font-bold py-2 px-4 rounded-lg transition duration-150 shadow-md text-sm">
                            Add Custom
                        </button>
                    </div>
                </div>
            </div>
            <!-- End Item Selection Panel -->

            <!-- Custom Modal/Notification Area -->
            <div id="modal-container" class="fixed inset-0 bg-gray-900 bg-opacity-50 hidden items-center justify-center z-50">
                <div class="bg-white p-6 rounded-lg shadow-xl w-80">
                    <p id="modal-message" class="text-gray-800 text-center mb-4 font-semibold"></p>
                    <div class="flex justify-center">
                        <button id="modal-confirm-btn" class="bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded-lg transition duration-150 mr-2">
                            Confirm Delete
                        </button>
                        <button id="modal-cancel-btn" class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-lg transition duration-150">
                            Cancel
                        </button>
                    </div>
                </div>
            </div>

            <p id="error-message" class="text-red-500 mt-4 hidden p-2 bg-red-100 border border-red-300 rounded-lg font-medium"></p>
            <p class="mt-6 text-sm text-gray-500">
                <span class="font-bold">Your User ID:</span> <span id="userIdDisplay">Loading...</span>
            </p>
        </div>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, onSnapshot, getDoc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- GLOBAL SETUP ---
        setLogLevel('Debug');
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let app;
        let db;
        let auth;
        let userId = 'anonymous';
        let isAuthReady = false; 

        // New data structure for items within a show
        // boothSpace: { selected: 'id', options: [...] }
        // buildingCost: { selected: 'id', options: [...] }
        // custom: [{ id: 'uuid', name: '...', cost: 0 }]
        let budgetData = {
            totalBudget: 100000,
            activeShowIds: [],
            editingShowId: '',
            shows: {}
        };

        const BUDGET_DOC_ID = 'tradeshowBudgets_v3'; // New ID due to data structure change

        // Predefined list of expense options derived from the CSV/Spreadsheet
        const expenseOptions = {
            boothSpace: [
                { id: '8x8', name: "8' x 8' Booth Space", category: 'boothSpace' },
                { id: '10x10', name: "10' x 10' Booth Space", category: 'boothSpace' },
                { id: '10x20', name: "10' x 20' Booth Space", category: 'boothSpace' },
            ],
            buildingCost: [
                { id: '10x10_bldg', name: "10' x 10' Booth Building Costs", category: 'buildingCost' },
                { id: '10x20_bldg', name: "10' x 20' Booth Building Costs", category: 'buildingCost' },
            ],
            // Custom items are managed separately in the show data
        };

        // Spreadsheet data structured for easy lookup and initialization
        const initialTradeshowsData = [
            { showName: 'AI Infra Summit, CA', costs: { '10x20': 23250, '10x20_bldg': 10000, '10x10_bldg': 5000 } },
            { showName: 'Embedded World NA, CA', costs: { '10x20': 9200, '10x10': 4600, '10x20_bldg': 10000, '10x10_bldg': 5000 } },
            { showName: 'Supercomputing, IL', costs: { '10x20': 10000, '10x10': 5000, '10x20_bldg': 10000, '10x10_bldg': 5000 } },
            { showName: 'Automate, IL', costs: { '10x20': 9600, '10x10': 4800, '10x20_bldg': 10000, '10x10_bldg': 5000 } },
            { showName: 'Robotics Summit, Boston', costs: { '10x20': 12200, '10x10': 5150, '10x20_bldg': 10000, '10x10_bldg': 5000 } },
            { showName: 'OCP Global Summit, CA', costs: { '8x8': 50000 } },
            { showName: 'Black Hat USA, NV', costs: { '10x10': 26500, '10x10_bldg': 8000 } },
            // UPDATED with 10x20 booth space cost of $14000
            { showName: 'Cyber Security & Cloud Congress at TechEx North America, CA', costs: { '10x10': 7000, '10x20': 14000, '10x20_bldg': 10000, '10x10_bldg': 5000 } },
        ];


        // --- Custom Modal/Notification Handling (instead of alert/confirm) ---
        const errorMessageEl = document.getElementById('error-message');
        const modalContainer = document.getElementById('modal-container');
        const modalMessage = document.getElementById('modal-message');
        const modalConfirmBtn = document.getElementById('modal-confirm-btn');
        const modalCancelBtn = document.getElementById('modal-cancel-btn');
        let currentDeleteResolve = null;

        function showErrorMessage(message) {
            errorMessageEl.textContent = message;
            errorMessageEl.classList.remove('hidden');
            setTimeout(() => errorMessageEl.classList.add('hidden'), 5000);
        }

        function showConfirmModal(message) {
            modalMessage.textContent = message;
            modalContainer.classList.remove('hidden');
            modalContainer.classList.add('flex');

            return new Promise((resolve) => {
                currentDeleteResolve = resolve;
            });
        }

        modalConfirmBtn.onclick = () => {
            if (currentDeleteResolve) {
                currentDeleteResolve(true);
                modalContainer.classList.remove('flex');
                modalContainer.classList.add('hidden');
                currentDeleteResolve = null;
            }
        };

        modalCancelBtn.onclick = () => {
            if (currentDeleteResolve) {
                currentDeleteResolve(false);
                modalContainer.classList.remove('flex');
                modalContainer.classList.add('hidden');
                currentDeleteResolve = null;
            }
        };

        // --- FIREBASE INITIALIZATION & AUTH ---
        try {
            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);
        } catch (e) {
            console.error("Firebase initialization failed:", e);
            showErrorMessage("Failed to initialize Firebase. Budget persistence will not work.");
        }

        const authListener = onAuthStateChanged(auth, async (user) => {
            if (user) {
                userId = user.uid;
            } else {
                userId = crypto.randomUUID(); 
            }
            document.getElementById('userIdDisplay').textContent = userId;
            isAuthReady = true; 

            if (db && isAuthReady) {
                await initializeFirestoreListener();
            }
            
            // Hide loading overlay once auth/data check is done
            document.getElementById('loading-overlay').classList.add('hidden');
        });

        async function authenticate() {
            try {
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (e) {
                console.error("Authentication failed:", e);
                showErrorMessage("Authentication failed. Cannot access saved budget data.");
            }
        }

        // --- FIRESTORE OPERATIONS ---

        function generateShowId(name) {
            // Simple slug-like ID generator
            return name.toLowerCase().replace(/[^a-z0-9]+/g, '_').replace(/^_|_$/g, '');
        }

        function initializeDefaultData() {
            const defaultData = {
                totalBudget: 100000,
                activeShowIds: [], 
                editingShowId: '',
                shows: {}
            };

            initialTradeshowsData.forEach(showData => {
                const id = generateShowId(showData.showName);
                
                // Construct the show's expense structure
                const showExpenses = {
                    boothSpace: { 
                        selected: Object.keys(showData.costs).find(k => k.includes('x') && !k.includes('bldg')) || null, // Pre-select the first available booth size
                        options: expenseOptions.boothSpace.map(opt => ({
                            ...opt, 
                            cost: showData.costs[opt.id] || 0, // Set cost based on show data
                            available: !!showData.costs[opt.id] // Mark as available if cost is non-zero
                        }))
                    },
                    buildingCost: {
                        selected: Object.keys(showData.costs).find(k => k.includes('bldg')) || null, // Pre-select the first available building cost
                        options: expenseOptions.buildingCost.map(opt => ({
                            ...opt, 
                            cost: showData.costs[opt.id] || 0, 
                            available: !!showData.costs[opt.id]
                        }))
                    },
                    custom: [] // Start with no custom items
                };

                defaultData.shows[id] = {
                    name: showData.showName,
                    expenses: showExpenses,
                };
            });

            // Set the first show as active and editing
            const allShowIds = Object.keys(defaultData.shows);
            const firstShowId = allShowIds[0] || '';
            defaultData.activeShowIds = firstShowId ? [firstShowId] : [];
            defaultData.editingShowId = firstShowId;
            
            return defaultData;
        }

        function getBudgetDocRef() {
            // Path: /artifacts/{appId}/public/data/tradeshowBudgets/tradeshowBudgets_v3
            return doc(db, `artifacts/${appId}/public/data/tradeshowBudgets`, BUDGET_DOC_ID);
        }

        async function saveBudgetData() {
            if (!db || !isAuthReady) {
                console.warn("Attempted to save budget data before authentication was ready.");
                return;
            }
            try {
                // We don't need extensive deep cloning/cleaning here since the data structure ensures costs are managed
                await setDoc(getBudgetDocRef(), budgetData);
            } catch (e) {
                console.error("Error saving budget data:", e);
                showErrorMessage("Failed to save budget data in real-time.");
            }
        }

        async function initializeFirestoreListener() {
            if (!db || !isAuthReady) return;

            const docRef = getBudgetDocRef();
            
            try {
                const docSnap = await getDoc(docRef);

                if (!docSnap.exists()) {
                    console.log("Budget document does not exist, setting default data.");
                    const defaultData = initializeDefaultData();
                    budgetData = defaultData;
                    await setDoc(docRef, defaultData);
                }
            } catch (e) {
                console.error("Error during initial document check/set:", e);
                showErrorMessage("Initial data load failed due to permissions or network issues.");
                return; 
            }

            onSnapshot(docRef, (docSnap) => {
                if (docSnap.exists()) {
                    const data = docSnap.data();

                    if (data.shows && Object.keys(data.shows).length > 0) {
                        budgetData = data;
                        budgetData.totalBudget = parseFloat(data.totalBudget) || 0;
                        
                        // Ensure activeShowIds is an array
                        budgetData.activeShowIds = Array.isArray(data.activeShowIds) ? data.activeShowIds : [];
                        
                        // Ensure editingShowId is valid, or set to a default
                        const allShowIds = Object.keys(budgetData.shows);
                        if (!budgetData.shows[data.editingShowId] && allShowIds.length > 0) {
                             budgetData.editingShowId = allShowIds[0];
                        } else if (!budgetData.shows[data.editingShowId] && allShowIds.length === 0) {
                            budgetData.editingShowId = '';
                        }
                    } else {
                        budgetData = initializeDefaultData();
                        saveBudgetData();
                    }
                    render();
                } else {
                    console.log("Budget document was deleted/not found, re-initializing with default.");
                    budgetData = initializeDefaultData();
                    render();
                }
            }, (error) => {
                console.error("Firestore listen error:", error);
                showErrorMessage("Real-time data synchronization failed.");
            });
        }

        // --- RENDERING & CALCULATIONS ---
        const totalBudgetInput = document.getElementById('totalBudgetInput');
        const allocatedBudgetEl = document.getElementById('allocatedBudget');
        const remainingBudgetEl = document.getElementById('remainingBudget');
        const budgetProgressBar = document.getElementById('budgetProgressBar');
        const statusMessageEl = document.getElementById('statusMessage');
        const activeShowCheckboxes = document.getElementById('activeShowCheckboxes');
        const editingShowSelector = document.getElementById('editingShowSelector');
        const showAllocatedEl = document.getElementById('showAllocated');
        const itemManagementPanel = document.getElementById('itemManagementPanel');
        const currentShowNameEl = document.getElementById('currentShowName');
        const boothSpaceOptionsEl = document.getElementById('boothSpaceOptions');
        const buildingCostOptionsEl = document.getElementById('buildingCostOptions');
        const customItemsListEl = document.getElementById('customItemsList');
        const noCustomItemsMessageEl = document.getElementById('noCustomItemsMessage');
        const boothSpaceCostEl = document.getElementById('boothSpaceCost');
        const buildingCostCostEl = document.getElementById('buildingCostCost');


        function formatCurrency(amount) {
            return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(amount);
        }

        /** Calculates the total allocated cost for a single tradeshow. */
        function calculateShowTotal(show) {
            if (!show || !show.expenses) return 0;
            let total = 0;

            // 1. Calculate selected Booth Space cost
            const bsSelectedId = show.expenses.boothSpace.selected;
            const bsOption = show.expenses.boothSpace.options.find(o => o.id === bsSelectedId);
            if (bsOption) total += parseFloat(bsOption.cost) || 0;

            // 2. Calculate selected Building Cost
            const bcSelectedId = show.expenses.buildingCost.selected;
            const bcOption = show.expenses.buildingCost.options.find(o => o.id === bcSelectedId);
            if (bcOption) total += parseFloat(bcOption.cost) || 0;

            // 3. Calculate Custom Items cost
            if (show.expenses.custom) {
                total += show.expenses.custom.reduce((sum, item) => sum + (parseFloat(item.cost) || 0), 0);
            }

            return total;
        }

        function calculateMetrics() {
            let totalAllocated = 0;
            const editingShowId = budgetData.editingShowId;
            let editingShowAllocated = 0;

            for (const id in budgetData.shows) {
                const show = budgetData.shows[id];
                const showTotal = calculateShowTotal(show);
                
                // Only include in GLOBAL total if it's an ACTIVE show
                if (budgetData.activeShowIds.includes(id)) {
                    totalAllocated += showTotal;
                }

                if (id === editingShowId) {
                    editingShowAllocated = showTotal;
                }
            }

            const remaining = budgetData.totalBudget - totalAllocated;
            const percentage = budgetData.totalBudget > 0 ? (totalAllocated / budgetData.totalBudget) * 100 : 0;

            return { totalAllocated, remaining, percentage: Math.min(percentage, 100), editingShowAllocated };
        }

        function renderTradeshowSelectors() {
            // 1. Render Checkboxes (Active Shows)
            activeShowCheckboxes.innerHTML = Object.entries(budgetData.shows).map(([id, show]) => {
                const isChecked = budgetData.activeShowIds.includes(id);
                return `
                    <label class="inline-flex items-center text-sm bg-white p-2 rounded-md border shadow-sm cursor-pointer hover:bg-indigo-50 transition duration-150">
                        <input type="checkbox" data-show-id="${id}" class="active-show-checkbox form-checkbox h-4 w-4 text-indigo-600 rounded" ${isChecked ? 'checked' : ''}>
                        <span class="ml-2 text-gray-700">${show.name}</span>
                    </label>
                `;
            }).join('');

            // 2. Render Single-Select Dropdown (Editing Show)
            editingShowSelector.innerHTML = Object.entries(budgetData.shows).map(([id, show]) => 
                `<option value="${id}">${show.name}</option>`
            ).join('');
            editingShowSelector.value = budgetData.editingShowId;

            // Re-attach checkbox listener
            document.querySelectorAll('.active-show-checkbox').forEach(checkbox => {
                checkbox.onchange = handleActiveShowChange;
            });
            editingShowSelector.onchange = handleEditingShowChange;
        }

        function renderItemManagementPanel(show) {
            const expenses = show.expenses;
            
            currentShowNameEl.textContent = show.name;
            itemManagementPanel.classList.remove('hidden');

            // --- A. BOOTH SPACE ---
            boothSpaceOptionsEl.innerHTML = expenses.boothSpace.options
                .filter(opt => opt.available)
                .map(opt => {
                    const isChecked = expenses.boothSpace.selected === opt.id;
                    const costString = formatCurrency(opt.cost);
                    return `
                        <label class="inline-flex items-center p-3 rounded-lg border cursor-pointer ${isChecked ? 'bg-indigo-100 border-indigo-500' : 'bg-white border-gray-300 hover:bg-gray-100'}">
                            <input type="radio" name="boothSpace" data-id="${opt.id}" class="booth-space-radio form-radio h-4 w-4 text-indigo-600" ${isChecked ? 'checked' : ''}>
                            <div class="ml-3 text-sm">
                                <span class="font-semibold text-gray-800">${opt.name}</span>
                                <span class="block text-xs font-medium text-indigo-700">${costString}</span>
                            </div>
                        </label>
                    `;
                }).join('');
            
            // Update booth space cost summary
            const bsSelected = expenses.boothSpace.options.find(o => o.id === expenses.boothSpace.selected);
            boothSpaceCostEl.textContent = bsSelected ? formatCurrency(bsSelected.cost) : 'N/A';
            
            // --- B. BUILDING COSTS ---
            buildingCostOptionsEl.innerHTML = expenses.buildingCost.options
                .filter(opt => opt.available)
                .map(opt => {
                    const isChecked = expenses.buildingCost.selected === opt.id;
                    const costString = formatCurrency(opt.cost);
                    return `
                        <label class="inline-flex items-center p-3 rounded-lg border cursor-pointer ${isChecked ? 'bg-indigo-100 border-indigo-500' : 'bg-white border-gray-300 hover:bg-gray-100'}">
                            <input type="radio" name="buildingCost" data-id="${opt.id}" class="building-cost-radio form-radio h-4 w-4 text-indigo-600" ${isChecked ? 'checked' : ''}>
                            <div class="ml-3 text-sm">
                                <span class="font-semibold text-gray-800">${opt.name}</span>
                                <span class="block text-xs font-medium text-indigo-700">${costString}</span>
                            </div>
                        </label>
                    `;
                }).join('');
            
            // Update building cost summary
            const bcSelected = expenses.buildingCost.options.find(o => o.id === expenses.buildingCost.selected);
            buildingCostCostEl.textContent = bcSelected ? formatCurrency(bcSelected.cost) : 'N/A';

            // --- C. CUSTOM ITEMS ---
            customItemsListEl.innerHTML = '';
            const customItems = expenses.custom || [];
            
            if (customItems.length === 0) {
                noCustomItemsMessageEl.classList.remove('hidden');
                customItemsListEl.appendChild(noCustomItemsMessageEl);
            } else {
                noCustomItemsMessageEl.classList.add('hidden');
                customItems.forEach(item => {
                    const itemEl = document.createElement('div');
                    itemEl.className = 'grid grid-cols-12 gap-4 items-center bg-gray-50 p-2 rounded-lg border border-gray-200';
                    itemEl.innerHTML = `
                        <div class="col-span-12 sm:col-span-6 font-medium text-gray-800">
                            <span class="sm:hidden text-xs font-semibold text-gray-500 block mb-1">Description</span>
                            <input type="text" data-id="${item.id}" data-field="name" value="${item.name}"
                                   class="w-full p-1 border rounded focus:ring-indigo-500 focus:border-indigo-500 text-xs">
                        </div>
                        <div class="col-span-8 sm:col-span-3 text-right">
                            <span class="sm:hidden text-xs font-semibold text-gray-500 block mb-1">Cost ($)</span>
                            <input type="number" data-id="${item.id}" data-field="cost" value="${(parseFloat(item.cost) || 0).toFixed(0)}" min="0"
                                   class="w-full p-1 border rounded focus:ring-indigo-500 focus:border-indigo-500 text-xs text-right">
                        </div>
                        <div class="col-span-4 sm:col-span-3 flex justify-end">
                            <button data-id="${item.id}" data-category="custom" class="delete-btn bg-red-500 hover:bg-red-600 text-white p-1 rounded-lg transition duration-150 shadow-sm text-xs font-semibold">
                                Delete
                            </button>
                        </div>
                    `;
                    customItemsListEl.appendChild(itemEl);
                });
            }

            // Re-attach listeners for the new item management panel
            attachItemManagementListeners();
        }


        function render() {
            const { totalAllocated, remaining, percentage, editingShowAllocated } = calculateMetrics();
            const currentShow = budgetData.shows[budgetData.editingShowId];

            // 1. Update Selectors (Active and Editing)
            renderTradeshowSelectors(); 

            // 2. Update Show-Specific Summary (using editingShowId)
            if (currentShow) {
                showAllocatedEl.textContent = `Total Budget Allocated for ${currentShow.name}: ${formatCurrency(editingShowAllocated)}`;
                renderItemManagementPanel(currentShow);
            } else {
                showAllocatedEl.textContent = 'Please select a show above to manage its budget items.';
                itemManagementPanel.classList.add('hidden');
            }


            // 3. Update Global Summary
            totalBudgetInput.value = budgetData.totalBudget.toFixed(0);
            allocatedBudgetEl.textContent = formatCurrency(totalAllocated);
            remainingBudgetEl.textContent = formatCurrency(remaining);

            // 4. Update Progress Bar & Status
            budgetProgressBar.style.width = `${percentage}%`;

            // Style updates based on budget status
            const isOverBudget = remaining < 0;
            const isCloseToBudget = percentage >= 75 && !isOverBudget;
            
            budgetProgressBar.classList.toggle('bg-red-600', isOverBudget);
            budgetProgressBar.classList.toggle('bg-yellow-500', isCloseToBudget);
            budgetProgressBar.classList.toggle('bg-indigo-600', !isOverBudget && !isCloseToBudget);

            remainingBudgetEl.parentElement.classList.toggle('bg-red-100', isOverBudget);
            remainingBudgetEl.parentElement.classList.toggle('border-red-300', isOverBudget);
            remainingBudgetEl.parentElement.classList.toggle('bg-yellow-50', isCloseToBudget);
            remainingBudgetEl.parentElement.classList.toggle('border-yellow-200', isCloseToBudget);
            remainingBudgetEl.parentElement.classList.toggle('bg-green-50', !isOverBudget && !isCloseToBudget);
            remainingBudgetEl.parentElement.classList.toggle('border-green-200', !isOverBudget && !isCloseToBudget);

            if (isOverBudget) {
                statusMessageEl.textContent = `OVER GLOBAL BUDGET by ${formatCurrency(Math.abs(remaining))}`;
                statusMessageEl.className = 'mt-2 text-sm text-center font-medium text-red-600';
            } else if (isCloseToBudget) {
                statusMessageEl.textContent = `Close to the global limit! ${percentage.toFixed(0)}% allocated.`;
                statusMessageEl.className = 'mt-2 text-sm text-center font-medium text-yellow-600';
            } else {
                statusMessageEl.textContent = `On track! ${percentage.toFixed(0)}% of global budget allocated.`;
                statusMessageEl.className = 'mt-2 text-sm text-center font-medium text-indigo-600';
            }
        }

        // --- EVENT HANDLERS ---
        function handleActiveShowChange(e) {
            const showId = e.target.dataset.showId;
            const isChecked = e.target.checked;
            
            if (isChecked && !budgetData.activeShowIds.includes(showId)) {
                budgetData.activeShowIds.push(showId);
            } else if (!isChecked) {
                budgetData.activeShowIds = budgetData.activeShowIds.filter(id => id !== showId);
            }

            // Ensure editingShowId remains valid if the current one is removed/set
            if (!budgetData.shows[budgetData.editingShowId]) {
                 budgetData.editingShowId = budgetData.activeShowIds[0] || Object.keys(budgetData.shows)[0] || '';
            }

            saveBudgetData(); 
        }

        function handleEditingShowChange(e) {
            budgetData.editingShowId = e.target.value;
            saveBudgetData(); 
            render(); 
        }

        function handleRadioSelection(category, selectedId) {
            const currentShow = budgetData.shows[budgetData.editingShowId];
            if (currentShow) {
                if (currentShow.expenses[category]) {
                    currentShow.expenses[category].selected = selectedId;
                    saveBudgetData();
                }
            }
        }

        function handleTotalBudgetChange(e) {
            const newBudget = parseFloat(e.target.value) || 0;
            if (budgetData.totalBudget !== newBudget) {
                budgetData.totalBudget = newBudget;
                saveBudgetData();
            }
        }
        
        function handleAddCustomItem() {
            const currentShow = budgetData.shows[budgetData.editingShowId];
            if (!currentShow) {
                showErrorMessage("Please select a tradeshow before adding a custom item.");
                return;
            }

            const nameInput = document.getElementById('newItemName');
            const costInput = document.getElementById('newItemCost');
            const name = nameInput.value.trim();
            const cost = parseFloat(costInput.value);

            if (!name) {
                showErrorMessage("Please enter an item description.");
                return;
            }
            if (isNaN(cost) || cost < 0) {
                showErrorMessage("Please enter a valid cost amount (zero or greater).");
                return;
            }

            const newItem = {
                id: crypto.randomUUID(),
                name: name,
                cost: cost
            };

            currentShow.expenses.custom.push(newItem);
            saveBudgetData();

            // Clear inputs
            nameInput.value = '';
            costInput.value = '';
            nameInput.focus();
        }

        async function handleDeleteCustomItem(itemId) {
            const confirmed = await showConfirmModal("Are you sure you want to delete this custom budget item?");
            if (confirmed) {
                const currentShow = budgetData.shows[budgetData.editingShowId];
                if (currentShow) {
                    currentShow.expenses.custom = currentShow.expenses.custom.filter(item => item.id !== itemId);
                    saveBudgetData();
                }
            }
        }

        function handleCustomItemFieldChange(e) {
            const input = e.target;
            const itemId = input.dataset.id;
            const field = input.dataset.field;
            let value = input.value;

            const currentShow = budgetData.shows[budgetData.editingShowId];
            if (!currentShow) return;

            const itemIndex = currentShow.expenses.custom.findIndex(item => item.id === itemId);
            if (itemIndex === -1) return;

            if (field === 'cost') {
                value = parseFloat(value) || 0;
                if (value < 0) value = 0;
            }

            if (currentShow.expenses.custom[itemIndex][field] !== value) {
                currentShow.expenses.custom[itemIndex][field] = value;
                saveBudgetData();
            }
        }

        function attachItemManagementListeners() {
            // Attach radio button listeners
            document.querySelectorAll('.booth-space-radio').forEach(radio => {
                radio.onchange = (e) => handleRadioSelection('boothSpace', e.target.dataset.id);
            });
            document.querySelectorAll('.building-cost-radio').forEach(radio => {
                radio.onchange = (e) => handleRadioSelection('buildingCost', e.target.dataset.id);
            });

            // Attach Custom Item listeners
            document.querySelectorAll('.delete-btn[data-category="custom"]').forEach(btn => {
                btn.onclick = () => handleDeleteCustomItem(btn.dataset.id);
            });
            document.querySelectorAll('#customItemsList input').forEach(input => {
                input.onblur = handleCustomItemFieldChange;
                input.onkeydown = (e) => {
                    if (e.key === 'Enter') {
                        e.target.blur();
                    }
                };
            });
        }

        // --- INITIAL SETUP ---
        document.getElementById('addItemBtn').onclick = handleAddCustomItem;
        totalBudgetInput.onblur = handleTotalBudgetChange;
        
        // Prevent form submission on enter for budget input
        totalBudgetInput.onkeydown = (e) => {
            if (e.key === 'Enter') {
                e.target.blur();
            }
        };

        // Start Auth process on load
        authenticate();

    </script>
</body>
</html># tradeshow2026
Trade Show Budget Allocator
