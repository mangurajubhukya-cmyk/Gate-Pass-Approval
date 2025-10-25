# Gate-Pass-Approval
Easy to enter or leave  with this websites
https://github.com/mangurajubhukya-cmyk/GatePass-Approval.git

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Secure Campus Pass - All-in-One</title>
    <!-- Load Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Poppins Font from Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    <!-- QR Code Library for generating the scannable pass -->
    <script src="https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js"></script>
    <style>
        /* Apply Poppins to everything */
        body {
            font-family: 'Poppins', sans-serif;
            background-color: #fcfcf5; /* Very light wheat color */
        }
        /* Custom scrollbar for better aesthetics */
        .custom-scroll::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scroll::-webkit-scrollbar-thumb {
            background-color: #03505d;
            border-radius: 3px;
        }
        /* Style for interactive buttons */
        .nav-btn {
            padding: 8px 16px;
            border-radius: 8px;
            font-weight: 600;
            transition: all 0.2s;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
        }
    </style>
    <script>
        // --- Tailwind Configuration with requested colors ---
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'primary': '#8f191d', /* Dark Red/Maroon */
                        'secondary': '#03505d', /* Dark Teal */
                        'wheat': '#fcfcf5',
                    }
                }
            }
        }
    </script>
</head>
<body class="min-h-screen flex flex-col antialiased text-gray-800">

    <div id="app-container" class="w-full max-w-7xl mx-auto p-4 sm:p-6 lg:p-8">

        <!-- Header and Navigation -->
        <header class="flex flex-col sm:flex-row justify-between items-start sm:items-center py-4 px-6 bg-white rounded-xl shadow-lg mb-8">
            <h1 class="text-3xl font-bold text-secondary mb-2 sm:mb-0">Secure Campus Pass</h1>
            <nav class="flex space-x-2 sm:space-x-4">
                <button onclick="setView('student')" id="nav-student" class="nav-btn bg-primary text-white hover:opacity-90">Student</button>
                <button onclick="setView('moderator')" id="nav-moderator" class="nav-btn bg-secondary text-white hover:opacity-90">Moderator</button>
                <button onclick="setView('gatekeeper')" id="nav-gatekeeper" class="nav-btn bg-secondary text-white hover:opacity-90">Gate Keeper</button>
            </nav>
        </header>

        <!-- Main Content Area -->
        <main class="bg-white rounded-xl shadow-2xl p-6 sm:p-10">
            <!-- Student View: Request Pass Form -->
            <div id="view-student" class="app-view hidden">
                <h2 class="text-2xl font-semibold mb-6 text-primary border-b-2 border-primary pb-2">Student Gate Pass Request</h2>
                <form id="student-form" class="space-y-6">
                    <div>
                        <label for="reason" class="block text-sm font-medium text-gray-700 mb-1">Reason for Leaving (Required)</label>
                        <textarea id="reason" rows="3" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-primary focus:border-primary transition duration-150"></textarea>
                    </div>
                    <div>
                        <label for="expectedReturn" class="block text-sm font-medium text-gray-700 mb-1">Expected Return Time (Required)</label>
                        <input type="datetime-local" id="expectedReturn" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-primary focus:border-primary transition duration-150">
                    </div>
                    <button type="submit" class="w-full sm:w-1/2 p-3 text-lg font-semibold text-white bg-primary rounded-lg shadow-md hover:shadow-lg transition duration-300 ease-in-out transform hover:-translate-y-0.5">
                        Submit Pass Request
                    </button>
                    <div id="student-message" class="mt-4 p-4 rounded-lg text-center hidden" role="alert"></div>
                    <div id="qr-code-display" class="mt-8 p-6 border-2 border-secondary border-dashed rounded-xl hidden text-center">
                        <p class="text-lg font-semibold text-secondary mb-4">Your Approved Pass QR Code</p>
                        <div id="qrcode-canvas" class="inline-block p-4 bg-wheat rounded-lg shadow-xl"></div>
                        <p class="mt-4 text-sm text-gray-600">Show this QR (or ID below) to the Gate Keeper.</p>
                        <code id="pass-id-code" class="mt-2 block text-sm font-mono text-primary bg-wheat p-2 rounded-md font-bold"></code>
                    </div>
                </form>
            </div>

            <!-- Moderator View: Pending Requests Dashboard -->
            <div id="view-moderator" class="app-view hidden">
                <h2 class="text-2xl font-semibold mb-6 text-primary border-b-2 border-primary pb-2">Moderator Approval Dashboard</h2>
                <div id="moderator-list-message" class="mt-4 p-4 rounded-lg text-center hidden" role="alert"></div>
                <div id="moderator-list" class="space-y-4 custom-scroll max-h-[70vh] overflow-y-auto mt-4">
                    <!-- Requests will be injected here -->
                    <p class="text-center text-gray-500 py-4 font-semibold">Loading requests...</p>
                </div>
            </div>

            <!-- Gate Keeper View: Scan/Check Pass -->
            <div id="view-gatekeeper" class="app-view hidden">
                <h2 class="text-2xl font-semibold mb-6 text-secondary border-b-2 border-secondary pb-2">Gate Keeper: Scan & Validate Pass</h2>
                <div class="max-w-md mx-auto space-y-4">
                    <!-- MODIFIED LABEL TO GUIDE USER TOWARDS STUDENT ID INPUT -->
                    <label for="passIdInput" class="block text-sm font-medium text-gray-700">Enter Student ID (STU-xxx) or Pass ID (ID-xxx)</label>
                    <input type="text" id="passIdInput" placeholder="Enter ID..." class="w-full p-3 border border-gray-300 rounded-lg focus:ring-secondary focus:border-secondary transition duration-150">
                    <button onclick="checkPassStatus()" class="w-full p-3 text-lg font-semibold text-white bg-secondary rounded-lg shadow-md hover:shadow-lg transition duration-300 ease-in-out transform hover:-translate-y-0.5">
                        Validate Pass
                    </button>
                    <div id="gatekeeper-result" class="mt-6 p-4 rounded-xl shadow-inner text-center hidden"></div>
                </div>
            </div>
        </main>

    </div>

<script>
    // --- FOUNDER-LEVEL IN-MEMORY DATABASE & ADVANCED TECH SIMULATION ---
    let database = []; // Simulates data.json
    let lastRequestedPassId = null; // Tracks the ID for the student to check status

    /**
     * Advanced Tech: Generates a secure-looking, unpredictable ID (UUID simulation).
     */
    const generateUUID = () => {
        return 'ID-' + Math.random().toString(36).substring(2, 6).toUpperCase() + Date.now().toString(36);
    };

    /**
     * Formats an ISO date string into a readable format (e.g., 25 Oct 2025 at 05:13 PM).
     */
    const formatDateTime = (isoString) => {
        const date = new Date(isoString);
        return date.toLocaleDateString('en-US', { day: '2-digit', month: 'short', year: 'numeric' }) + ' at ' + date.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', hour12: true });
    }

    // --- Utility Functions ---

    /**
     * Displays a temporary message to the user.
     */
    function showMessage(element, message, type) {
        element.textContent = message;
        element.classList.remove('hidden', 'bg-red-100', 'text-red-700', 'bg-green-100', 'text-green-700', 'bg-yellow-100', 'text-yellow-700');

        if (type === 'success') {
            element.classList.add('bg-green-100', 'text-green-700');
        } else if (type === 'error') {
            element.classList.add('bg-red-100', 'text-red-700');
        } else { // info/pending
            element.classList.add('bg-yellow-100', 'text-yellow-700');
        }

        setTimeout(() => {
            element.classList.add('hidden');
        }, 8000); // Increased visibility time
        element.classList.remove('hidden');
    }

    /**
     * Switches the main application view (Student, Moderator, Gate Keeper).
     */
    function setView(viewName) {
        // Hide all views
        document.querySelectorAll('.app-view').forEach(view => {
            view.classList.add('hidden');
        });

        // Show the selected view
        document.getElementById(`view-${viewName}`).classList.remove('hidden');

        // Update nav buttons style
        document.getElementById('nav-student').classList.remove('bg-primary', 'bg-secondary');
        document.getElementById('nav-moderator').classList.remove('bg-primary', 'bg-secondary');
        document.getElementById('nav-gatekeeper').classList.remove('bg-primary', 'bg-secondary');

        // Set active button to primary color
        document.getElementById(`nav-${viewName}`).classList.add('bg-primary');

        // Set inactive buttons to secondary color
        if (viewName !== 'student') document.getElementById('nav-student').classList.add('bg-secondary');
        if (viewName !== 'moderator') document.getElementById('nav-moderator').classList.add('bg-secondary');
        if (viewName !== 'gatekeeper') document.getElementById('nav-gatekeeper').classList.add('bg-secondary');


        // Specific actions on view change
        if (viewName === 'moderator') {
            renderModeratorList();
        } else if (viewName === 'student' && lastRequestedPassId) {
            checkLastPassStatus(lastRequestedPassId); // Student checks if their pass got approved
        }
    }

    // --- 1. Student Logic ---

    document.getElementById('student-form').addEventListener('submit', (e) => {
        e.preventDefault();
        const reason = document.getElementById('reason').value;
        const expectedReturn = document.getElementById('expectedReturn').value;
        const messageBox = document.getElementById('student-message');
        const qrDisplay = document.getElementById('qr-code-display');

        // 1. Create the new pass object
        const newPass = {
            id: generateUUID(),
            studentId: 'STU' + Math.floor(Math.random() * 900 + 100), // Mock ID (for all students)
            reason: reason,
            expectedReturn: expectedReturn,
            timestamp: new Date().toISOString(),
            status: 'PENDING', // PENDING, ACCEPTED, REJECTED, USED
            moderatorRemarks: null,
        };

        // 2. Add to in-memory database
        database.push(newPass);
        lastRequestedPassId = newPass.id; // Store for student's immediate status check

        // 3. Update UI
        showMessage(messageBox, `Request submitted! Your Student ID is: ${newPass.studentId}. Temporary Pass ID: ${newPass.id}. Check back later to see if it's approved.`, 'info');
        document.getElementById('student-form').reset();
        qrDisplay.classList.add('hidden'); // Hide QR code until approval
    });

    /**
     * Checks the status of the student's last requested pass.
     */
    function checkLastPassStatus(passId) {
        if (!passId) return;

        const pass = database.find(p => p.id === passId);
        const statusBox = document.getElementById('student-message');
        const qrDisplay = document.getElementById('qr-code-display');
        const qrCanvas = document.getElementById('qrcode-canvas');
        const passIdCode = document.getElementById('pass-id-code');

        if (!pass) {
            showMessage(statusBox, 'Could not find your last requested pass ID.', 'error');
            qrDisplay.classList.add('hidden');
            return;
        }

        if (pass.status === 'ACCEPTED') {
            showMessage(statusBox, 'PASS APPROVED! Your QR code is ready below. Show it to the Gate Keeper.', 'success');
            qrDisplay.classList.remove('hidden');
            passIdCode.textContent = passId;

            // Advanced Tech: QR Code Generation
            qrCanvas.innerHTML = ''; // Clear previous QR
            new QRCode(qrCanvas, {
                text: passId, // The Gate Keeper scans this ID
                width: 150,
                height: 150,
                colorDark: "#03505d",
                colorLight: "#fcfcf5",
                correctLevel: QRCode.CorrectLevel.H
            });
        } else if (pass.status === 'PENDING') {
            showMessage(statusBox, 'Your pass is PENDING approval. Please wait for a moderator.', 'info');
            qrDisplay.classList.add('hidden');
        } else if (pass.status === 'REJECTED') {
            showMessage(statusBox, `PASS REJECTED: ${pass.moderatorRemarks || 'No remarks provided.'}`, 'error');
            qrDisplay.classList.add('hidden');
        } else if (pass.status === 'USED') {
            showMessage(statusBox, 'PASS USED: This pass has been validated by the Gate Keeper and is no longer active.', 'error');
            qrDisplay.classList.add('hidden');
        }
    }


    // --- 2. Moderator Logic ---

    function renderModeratorList() {
        const listContainer = document.getElementById('moderator-list');
        listContainer.innerHTML = ''; // Clear existing list
        const messageBox = document.getElementById('moderator-list-message');

        const pendingRequests = database.filter(p => p.status === 'PENDING');
        if (pendingRequests.length === 0) {
            listContainer.innerHTML = '<p class="text-center text-secondary py-4 font-semibold">No pending gate pass requests found!</p>';
        }

        // Sort by timestamp (newest first)
        const sortedRequests = [...database].sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));

        sortedRequests.forEach(pass => {
            const isPending = pass.status === 'PENDING';

            const card = document.createElement('div');
            card.id = `pass-card-${pass.id}`;
            card.className = `p-4 rounded-xl shadow-md transition duration-300 ease-in-out border-l-4 ${
                isPending ? 'bg-yellow-50 border-yellow-500' :
                (pass.status === 'ACCEPTED' ? 'bg-green-50 border-green-500' :
                (pass.status === 'USED' ? 'bg-blue-50 border-blue-500' :
                'bg-red-50 border-red-500'))
            }`;

            card.innerHTML = `
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center border-b pb-2 mb-2 border-gray-200">
                    <span class="text-lg font-bold text-secondary">Pass ID: ${pass.id}</span>
                    <span class="text-sm font-semibold text-gray-600">Student ID: ${pass.studentId}</span>
                </div>
                <p class="text-sm"><span class="font-medium text-gray-700">Status:</span>
                    <span class="font-bold p-1 rounded-md text-white ${pass.status === 'PENDING' ? 'bg-yellow-500' : (pass.status === 'ACCEPTED' ? 'bg-green-600' : (pass.status === 'USED' ? 'bg-blue-600' : 'bg-red-600'))}">${pass.status}</span>
                </p>
                <p class="mt-1 text-sm"><span class="font-medium text-gray-700">Reason:</span> ${pass.reason}</p>
                <p class="text-sm"><span class="font-medium text-gray-700">Return By:</span> ${formatDateTime(pass.expectedReturn)}</p>
                ${pass.moderatorRemarks ? `<p class="mt-2 text-sm text-gray-700 border-l-4 border-primary pl-2 italic">Remarks: ${pass.moderatorRemarks}</p>` : ''}

                ${isPending ? `
                    <div class="mt-4 pt-4 border-t border-gray-200 flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-4">
                        <input type="text" id="remarks-${pass.id}" placeholder="Remarks (Required for Rejection, min 5 chars)" class="flex-grow p-2 border border-gray-300 rounded-lg text-sm">
                        <button onclick="handleApproval('${pass.id}', 'ACCEPTED')" class="bg-green-600 text-white p-2 rounded-lg text-sm font-semibold hover:bg-green-700 transition duration-150">Accept</button>
                        <button onclick="handleApproval('${pass.id}', 'REJECTED')" class="bg-red-600 text-white p-2 rounded-lg text-sm font-semibold hover:bg-red-700 transition duration-150">Reject</button>
                    </div>
                ` : ''}
            `;
            listContainer.appendChild(card);
        });
    }

    function handleApproval(passId, status) {
        const passIndex = database.findIndex(p => p.id === passId);
        const messageBox = document.getElementById('moderator-list-message');

        if (passIndex === -1 || database[passIndex].status !== 'PENDING') {
            showMessage(messageBox, `Error: Pass ${passId} not found or not pending.`, 'error');
            return;
        }

        const remarksInput = document.getElementById(`remarks-${passId}`);
        const remarks = remarksInput ? remarksInput.value.trim() : '';

        // Enforce reason for REJECTION (minimum 5 characters)
        if (status === 'REJECTED' && remarks.length < 5) {
             showMessage(messageBox, 'Rejection requires a clear reason (at least 5 characters).', 'error');
             return;
        }

        // Update in-memory database
        database[passIndex].status = status;
        database[passIndex].moderatorRemarks = remarks || (status === 'REJECTED' ? 'Rejected by Moderator.' : 'Approved.');
        database[passIndex].moderatorTime = new Date().toISOString();

        // Show confirmation and re-render
        showMessage(messageBox, `Pass ${passId} has been successfully ${status}.`, 'success');
        renderModeratorList();

        // If the student is viewing their pass, update their status display
        if (lastRequestedPassId === passId) {
             checkLastPassStatus(passId);
        }
    }

    // --- 3. Gate Keeper Logic ---

    function checkPassStatus() {
        const inputId = document.getElementById('passIdInput').value.trim();
        const resultBox = document.getElementById('gatekeeper-result');
        resultBox.classList.add('hidden');
        resultBox.innerHTML = '';

        if (!inputId) {
            showMessage(resultBox, 'Please enter a Student ID or Pass ID to check.', 'error');
            return;
        }

        // 1. Find all passes that match either Pass ID or Student ID
        let passes = database.filter(p => p.id === inputId || p.studentId === inputId);

        if (passes.length === 0) {
            resultBox.classList.remove('hidden');
            resultBox.className = 'mt-6 p-6 rounded-xl shadow-lg text-center bg-red-200 border-2 border-red-700 text-red-700';
            resultBox.innerHTML = `<div class="text-xl font-bold">ERROR: ID "${inputId}" Not Found.</div>`;
            return;
        }

        // 2. Sort by timestamp (newest first)
        passes.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));

        // 3. Find the most relevant, active pass: ACCEPTED, PENDING, or the latest one for history display
        let pass = passes.find(p => p.status === 'ACCEPTED') ||
                   passes.find(p => p.status === 'PENDING') ||
                   passes[0]; 

        // 4. Validate status and update
        let statusColor;
        let statusMessage;

        if (pass.status === 'ACCEPTED') {
            // Status Transition: Mark as used immediately upon successful scan
            pass.status = 'USED'; 
            statusColor = 'bg-green-200 border-green-700 text-green-700';
            statusMessage = 'PASS VALIDATED: Entry/Exit Allowed. Status updated to USED.';
        } else if (pass.status === 'USED') {
            statusColor = 'bg-yellow-200 border-yellow-700 text-yellow-700';
            statusMessage = 'PASS ALREADY USED: This pass is inactive.';
        } else if (pass.status === 'PENDING') {
            statusColor = 'bg-yellow-200 border-orange-700 text-orange-700';
            statusMessage = 'PASS PENDING: Approval required before use. Student cannot leave.';
        } else { // REJECTED
            statusColor = 'bg-red-200 border-red-700 text-red-700';
            statusMessage = 'PASS REJECTED: Entry/Exit Forbidden.';
        }

        // 5. Display Result
        resultBox.classList.remove('hidden');
        resultBox.className = `mt-6 p-6 rounded-xl shadow-lg text-left border-2 ${statusColor}`;
        resultBox.innerHTML = `
            <div class="text-2xl font-bold mb-4">${statusMessage}</div>
            <p class="text-sm"><span class="font-semibold">Pass ID:</span> ${pass.id}</p>
            <p class="text-sm"><span class="font-semibold">Student ID:</span> ${pass.studentId}</p>
            <p class="text-sm"><span class="font-semibold">Reason:</span> ${pass.reason}</p>
            <p class="text-sm"><span class="font-semibold">Return By:</span> ${formatDateTime(pass.expectedReturn)}</p>
            <p class="mt-2 text-sm border-t pt-2 border-gray-400"><span class="font-semibold">Moderator Notes:</span> ${pass.moderatorRemarks || 'None'}</p>
        `;

        // 6. Global updates if status changed
        if (pass.status === 'USED') {
            renderModeratorList();
            if (lastRequestedPassId === pass.id) {
                checkLastPassStatus(pass.id);
            }
        }
    }


    // --- Initialization ---
    window.onload = () => {
        // Initialize with multiple mock requests to demonstrate the system for all students.
        database.push({
            id: 'MOCK-1234',
            studentId: 'STU-001',
            reason: 'Urgent meeting with academic advisor off-campus.',
            expectedReturn: new Date(Date.now() + 3600000).toISOString(), // 1 hour from now
            timestamp: new Date().toISOString(),
            status: 'PENDING',
            moderatorRemarks: null,
        });
        database.push({
            id: 'MOCK-5678',
            studentId: 'STU-002',
            reason: 'Family emergency, must leave immediately.',
            expectedReturn: new Date(Date.now() + 7200000).toISOString(), // 2 hours from now
            timestamp: new Date(Date.now() - 60000).toISOString(), // Submitted 1 minute earlier
            status: 'PENDING',
            moderatorRemarks: null,
        });
        database.push({
            id: 'MOCK-REJ',
            studentId: 'STU-003',
            reason: 'Going to the mall to shop for new clothes.',
            expectedReturn: new Date(Date.now() + 10800000).toISOString(), 
            timestamp: new Date(Date.now() - 120000).toISOString(), 
            status: 'REJECTED',
            moderatorRemarks: 'Non-academic/non-emergency reason for absence.',
        });
        // A pass that is already ACCEPTED for testing direct validation
        database.push({
            id: 'MOCK-ACC',
            studentId: 'STU-004',
            reason: 'External sports competition.',
            expectedReturn: new Date(Date.now() + 4800000).toISOString(), 
            timestamp: new Date(Date.now() - 180000).toISOString(), 
            status: 'ACCEPTED',
            moderatorRemarks: 'Verified by coach.',
        });
        setView('student'); // Start on the Student view
    };
</script>
</body>
</html>
