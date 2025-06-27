<html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Draw the Verb</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: "Inter", sans-serif; /* Inter Font */
        }
        /* Canvas styling */
        #drawingCanvas {
            border: 2px solid #3B82F6; /* Tailwind Blue 600 */
            background-color: #F8FAFC; /* Tailwind Soft White */
            cursor: crosshair;
            touch-action: none; /* Prevents browser scrolling when drawing */
        }
        /* Style for the emoji container */
        #verbEmoji {
            font-size: 10rem; /* Large size for the emoji */
            line-height: 1;
            margin-bottom: 1rem;
            display: flex;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 250px; /* Maintain similar height as the image placeholder */
            background-color: #E0E7FF; /* Light blue background for consistency */
            border-radius: 0.75rem; /* rounded-xl */
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06); /* shadow-lg */
        }
    </style>
</head>
<body class="bg-gradient-to-r from-blue-400 to-purple-500 min-h-screen flex items-center justify-center p-4">
    <div class="bg-white p-8 rounded-2xl shadow-xl max-w-md w-full text-center">
        <h1 class="text-4xl font-extrabold text-gray-800 mb-6">Draw the Verb</h1>

        <!-- Container for the flashcard and the verb -->
        <div class="flex flex-col items-center justify-center mb-8">
            <!-- Verb Emoji -->
            <div id="verbEmoji" class="relative rounded-xl shadow-lg mb-4">
                <!-- Emoji will be displayed here -->
            </div>

            <div class="flex items-center space-x-4 mb-6 mt-4">
                <!-- Current verb -->
                <p id="currentVerb" class="text-6xl font-bold text-gray-900">Loading...</p>
                <!-- Pronounce button -->
                <button id="pronounceButton" class="bg-blue-600 hover:bg-blue-700 text-white p-3 rounded-full shadow-lg transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-blue-300">
                    <span class="text-3xl">🔊</span>
                </button>
                <!-- Pronunciation -->
                <p id="pronunciation" class="text-4xl text-gray-600">...</p>
            </div>
        </div>

        <!-- Instructions for the user -->
        <p class="text-xl text-gray-700 mb-8">
            Draw this action here:
        </p>

        <!-- Drawing area with canvas -->
        <div class="mb-8 flex flex-col items-center">
            <canvas id="drawingCanvas" width="300" height="200" class="rounded-lg shadow-md w-full max-w-xs mb-4"></canvas>
            <div class="flex space-x-4">
                <button id="clearCanvasButton" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-full shadow-md transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-red-300">
                    Clear Drawing
                </button>
                <button id="drawnButton" class="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-full shadow-md transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-green-300">
                    I've Drawn It!
                </button>
            </div>
        </div>

        <!-- Continue button -->
        <button id="continueButton" class="bg-purple-600 hover:bg-purple-700 text-white font-bold py-3 px-10 rounded-full shadow-lg transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-purple-300">
            Continue
        </button>
    </div>

    <script>
        // Array of verbs with their pronunciation and corresponding emoji
        const verbs = [
            { verb: "sing", pronunciation: "/sɪŋ/", emoji: "🎤" },
            { verb: "read", pronunciation: "/riːd/", emoji: "📚" },
            { verb: "dance", pronunciation: "/dæns/", emoji: "💃" },
            { verb: "run", pronunciation: "/rʌn/", emoji: "🏃‍♂️" },
            { verb: "jump", pronunciation: "/dʒʌmp/", emoji: "🤸‍♂️" }
        ];

        let currentVerbIndex = 0;

        const verbEmojiElement = document.getElementById('verbEmoji'); // Element to display the emoji
        const currentVerbElement = document.getElementById('currentVerb');
        const pronunciationElement = document.getElementById('pronunciation');
        const pronounceButton = document.getElementById('pronounceButton');
        const drawnButton = document.getElementById('drawnButton');
        const continueButton = document.getElementById('continueButton');

        // Canvas elements
        const drawingCanvas = document.getElementById('drawingCanvas');
        const ctx = drawingCanvas.getContext('2d');
        const clearCanvasButton = document.getElementById('clearCanvasButton');

        let isDrawing = false;
        let lastX = 0;
        let lastY = 0;

        /**
         * Adjusts the canvas size to be responsive.
         */
        function resizeCanvas() {
            // Get the computed style of the canvas's parent to determine available width
            const parentWidth = drawingCanvas.parentElement.offsetWidth;
            drawingCanvas.width = Math.min(parentWidth, 300); // Max width of 300px as per current max-w-xs
            drawingCanvas.height = 200; // Keep height fixed for now

            // Re-apply drawing styles after resize
            ctx.lineWidth = 2;
            ctx.lineJoin = 'round';
            ctx.lineCap = 'round';
            ctx.strokeStyle = '#000'; // Default drawing color
        }

        // Call resize on initial load and on window resize
        window.addEventListener('load', resizeCanvas);
        window.addEventListener('resize', resizeCanvas);

        /**
         * Starts drawing.
         * @param {Event} e - Mouse or touch event.
         */
        function startDrawing(e) {
            isDrawing = true;
            [lastX, lastY] = getCoordinates(e);
        }

        /**
         * Draws a line on the canvas.
         * @param {Event} e - Mouse or touch event.
         */
        function draw(e) {
            if (!isDrawing) return;

            const [x, y] = getCoordinates(e);

            ctx.beginPath();
            ctx.moveTo(lastX, lastY);
            ctx.lineTo(x, y);
            ctx.stroke();
            [lastX, lastY] = [x, y];
        }

        /**
         * Stops drawing.
         */
        function stopDrawing() {
            isDrawing = false;
        }

        /**
         * Gets mouse or touch coordinates relative to the canvas.
         * @param {Event} e - Mouse or touch event.
         * @returns {Array<number>} [x, y] coordinates.
         */
        function getCoordinates(e) {
            const rect = drawingCanvas.getBoundingClientRect();
            let clientX, clientY;

            if (e.touches && e.touches.length > 0) {
                clientX = e.touches[0].clientX;
                clientY = e.touches[0].clientY;
            } else {
                clientX = e.clientX;
                clientY = e.clientY;
            }

            return [clientX - rect.left, clientY - rect.top];
        }

        // Event listeners for drawing
        drawingCanvas.addEventListener('mousedown', startDrawing);
        drawingCanvas.addEventListener('mousemove', draw);
        drawingCanvas.addEventListener('mouseup', stopDrawing);
        drawingCanvas.addEventListener('mouseout', stopDrawing); // Stop drawing if mouse leaves canvas

        // Touch events for drawing
        drawingCanvas.addEventListener('touchstart', startDrawing);
        drawingCanvas.addEventListener('touchmove', draw);
        drawingCanvas.addEventListener('touchend', stopDrawing);
        drawingCanvas.addEventListener('touchcancel', stopDrawing);

        /**
         * Clears the canvas content.
         */
        clearCanvasButton.addEventListener('click', () => {
            ctx.clearRect(0, 0, drawingCanvas.width, drawingCanvas.height);
        });

        /**
         * Displays the current verb and its emoji.
         */
        function displayCurrentVerb() {
            const verbData = verbs[currentVerbIndex];
            currentVerbElement.textContent = verbData.verb;
            pronunciationElement.textContent = verbData.pronunciation;
            verbEmojiElement.textContent = verbData.emoji; // Set the emoji

            // Clear the canvas for the new verb
            ctx.clearRect(0, 0, drawingCanvas.width, drawingCanvas.height);
        }

        /**
         * Plays the pronunciation of the current verb.
         */
        pronounceButton.addEventListener('click', () => {
            const verbToPronounce = currentVerbElement.textContent;
            console.log(`Pronouncing: ${verbToPronounce}`);
            const utterance = new SpeechSynthesisUtterance(verbToPronounce);
            utterance.lang = 'en-US'; // Set language for correct pronunciation
            window.speechSynthesis.speak(utterance);
        });

        /**
         * Handles the click on the "I've Drawn It!" button.
         */
        drawnButton.addEventListener('click', () => {
            console.log("User has finished drawing!");
            drawnButton.textContent = "Drawing Completed!";
            drawnButton.classList.remove('bg-green-500', 'hover:bg-green-600');
            drawnButton.classList.add('bg-gray-400');
            setTimeout(() => {
                drawnButton.textContent = "I've Drawn It!";
                drawnButton.classList.remove('bg-gray-400');
                drawnButton.classList.add('bg-green-500', 'hover:bg-green-600');
            }, 1500);
        });

        /**
         * Advances to the next verb or ends the activity.
         */
        continueButton.addEventListener('click', () => {
            currentVerbIndex++;
            if (currentVerbIndex < verbs.length) {
                displayCurrentVerb();
            } else {
                currentVerbElement.textContent = "Activity Complete!";
                pronunciationElement.textContent = "";
                verbEmojiElement.textContent = "🎉"; // Final emoji
                // Disable drawing functionality
                drawingCanvas.removeEventListener('mousedown', startDrawing);
                drawingCanvas.removeEventListener('mousemove', draw);
                drawingCanvas.removeEventListener('mouseup', stopDrawing);
                drawingCanvas.removeEventListener('mouseout', stopDrawing);
                drawingCanvas.removeEventListener('touchstart', startDrawing);
                drawingCanvas.removeEventListener('touchmove', draw);
                drawingCanvas.removeEventListener('touchend', stopDrawing);
                drawingCanvas.removeEventListener('touchcancel', stopDrawing);
                clearCanvasButton.disabled = true;

                pronounceButton.disabled = true;
                drawnButton.disabled = true;
                continueButton.disabled = true;
                continueButton.textContent = "End";
                console.log("All verbs have been displayed.");
            }
        });

        // Initialize the activity when the page loads
        displayCurrentVerb();
    </script>
</body>
</html>
ELABORADO POR: MARÍA PAZ PERALTA
