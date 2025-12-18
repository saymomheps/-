# -
গহহহত
<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI রিয়েলিস্টিক ইমেজ জেনারেটর</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Hind+Siliguri:wght@300;400;600&display=swap');
        
        body {
            font-family: 'Hind Siliguri', sans-serif;
            background-color: #0f172a;
            color: #f8fafc;
        }
        
        .glass-morphism {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #3b82f6;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .image-container img {
            max-width: 100%;
            height: auto;
            border-radius: 12px;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.3);
            transition: transform 0.3s ease;
        }

        .image-container img:hover {
            transform: scale(1.01);
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center p-4 md:p-8">

    <div class="max-w-4xl w-full space-y-8">
        <!-- Header -->
        <div class="text-center space-y-4">
            <h1 class="text-4xl md:text-5xl font-bold bg-gradient-to-r from-blue-400 to-emerald-400 bg-clip-text text-transparent">
                সরাসরি প্রম্পট ইমেজ জেনারেটর
            </h1>
            <p class="text-slate-400">আপনি যা লিখবেন, ঠিক সেটিরই ছবি তৈরি হবে</p>
        </div>

        <!-- Input Area -->
        <div class="glass-morphism p-6 rounded-2xl shadow-2xl space-y-6">
            <div class="space-y-2">
                <label for="prompt" class="block text-sm font-medium text-slate-300">আপনার নিখুঁত প্রম্পটটি এখানে লিখুন:</label>
                <textarea 
                    id="prompt" 
                    rows="3" 
                    placeholder="হুবহু যা চান তা এখানে লিখুন..."
                    class="w-full p-4 rounded-xl bg-slate-900 border border-slate-700 text-white focus:ring-2 focus:ring-blue-500 focus:border-transparent outline-none transition-all"
                ></textarea>
            </div>

            <div class="flex flex-col md:flex-row gap-4 items-center">
                <button 
                    id="generateBtn"
                    class="w-full md:w-auto px-8 py-3 bg-blue-600 hover:bg-blue-700 text-white font-semibold rounded-xl flex items-center justify-center gap-2 transition-all active:scale-95"
                >
                    <i class="fas fa-magic"></i> ছবি তৈরি করুন
                </button>
                <p id="status" class="text-sm text-slate-400 italic hidden">প্রসেসিং হচ্ছে...</p>
            </div>
        </div>

        <!-- Output Area -->
        <div id="outputSection" class="hidden glass-morphism p-4 rounded-2xl overflow-hidden min-h-[400px] flex flex-col items-center justify-center">
            <div id="loadingIndicator" class="hidden flex flex-col items-center gap-4">
                <div class="loader"></div>
                <p>ছবিটি তৈরি হচ্ছে...</p>
            </div>
            
            <div id="imageDisplay" class="image-container w-full flex flex-col items-center gap-6">
                <!-- Image output -->
            </div>
        </div>
    </div>

    <script>
        const apiKey = ""; // API key is handled by the environment
        const generateBtn = document.getElementById('generateBtn');
        const promptInput = document.getElementById('prompt');
        const outputSection = document.getElementById('outputSection');
        const loadingIndicator = document.getElementById('loadingIndicator');
        const imageDisplay = document.getElementById('imageDisplay');
        const statusText = document.getElementById('status');

        async function generateImage() {
            const promptValue = promptInput.value.trim();
            if (!promptValue) {
                alert("দয়া করে একটি প্রম্পট লিখুন!");
                return;
            }

            // UI Reset
            generateBtn.disabled = true;
            statusText.classList.remove('hidden');
            outputSection.classList.remove('hidden');
            loadingIndicator.classList.remove('hidden');
            imageDisplay.innerHTML = "";

            let retries = 0;
            const maxRetries = 5;

            async function attemptFetch() {
                try {
                    const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:predict?key=${apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({
                            instances: [{ prompt: promptValue }], // সরাসরি ইউজার প্রম্পট ব্যবহার করা হচ্ছে
                            parameters: { sampleCount: 1 }
                        })
                    });

                    if (!response.ok) throw new Error('API Error');

                    const result = await response.json();
                    if (result.predictions && result.predictions[0].bytesBase64Encoded) {
                        const imageUrl = `data:image/png;base64,${result.predictions[0].bytesBase64Encoded}`;
                        displayResult(imageUrl);
                    } else {
                        throw new Error('No image found');
                    }
                } catch (error) {
                    if (retries < maxRetries) {
                        retries++;
                        const delay = Math.pow(2, retries) * 1000;
                        setTimeout(attemptFetch, delay);
                    } else {
                        handleError();
                    }
                }
            }

            attemptFetch();
        }

        function displayResult(url) {
            loadingIndicator.classList.add('hidden');
            generateBtn.disabled = false;
            statusText.classList.add('hidden');

            const img = document.createElement('img');
            img.src = url;
            img.alt = "Generated Image";
            img.className = "max-w-full rounded-lg shadow-2xl";

            const downloadBtn = document.createElement('button');
            downloadBtn.innerHTML = '<i class="fas fa-download"></i> ইমেজ ডাউনলোড';
            downloadBtn.className = "px-6 py-2 bg-emerald-600 hover:bg-emerald-700 text-white rounded-lg transition-all";
            downloadBtn.onclick = () => {
                const link = document.createElement('a');
                link.href = url;
                link.download = 'ai-image.png';
                link.click();
            };

            imageDisplay.appendChild(img);
            imageDisplay.appendChild(downloadBtn);
        }

        function handleError() {
            loadingIndicator.classList.add('hidden');
            generateBtn.disabled = false;
            statusText.classList.add('hidden');
            imageDisplay.innerHTML = `
                <div class="text-center p-8 text-red-400">
                    <p>ছবিটি তৈরি করা সম্ভব হয়নি। আবার চেষ্টা করুন।</p>
                </div>
            `;
        }

        generateBtn.addEventListener('click', generateImage);
        promptInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                generateImage();
            }
        });
    </script>
</body>
</html>
