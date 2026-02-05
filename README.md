<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PhodioEdit - Edit Photos and Videos</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/fabric.js/5.3.0/fabric.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background: #f4f4f4; }
        #auth { display: block; }
        #editor { display: none; }
        .container { max-width: 800px; margin: auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        input, button { margin: 10px 0; padding: 10px; width: 100%; }
        canvas { border: 1px solid #ccc; }
        video { width: 100%; }
        .tools { margin-top: 20px; }
        .tools button { width: auto; margin-right: 10px; }
        .subscription { margin-top: 20px; border-top: 1px solid #ccc; padding-top: 20px; }
        .subscription button { background: #28a745; color: white; border: none; }
        .subscription button:hover { background: #218838; }
        .premium { display: none; } /* Hidden by default, shown if subscribed */
    </style>
</head>
<body>
    <div id="auth" class="container">
        <h1>PhodioEdit</h1>
        <h2>Sign In or Sign Up</h2>
        <input type="text" id="name" placeholder="Name" required>
        <input type="email" id="email" placeholder="Email" required>
        <input type="password" id="password" placeholder="Password" required>
        <button onclick="signUp()">Sign Up</button>
        <button onclick="signIn()">Sign In</button>
        <p id="authMessage"></p>
    </div>

    <div id="editor" class="container">
        <h1>Welcome to PhodioEdit, <span id="userName"></span>!</h1>
        <button onclick="logout()">Logout</button>
        
        <div class="subscription">
            <h2>Subscription Plans</h2>
            <p>Unlock premium filters and effects!</p>
            <button onclick="subscribe(1)">1 Month - ₹110</button>
            <button onclick="subscribe(3)">3 Months - ₹250</button>
            <p id="subMessage"></p>
        </div>
        
        <h2>Photo Editor</h2>
        <input type="file" id="photoUpload" accept="image/*">
        <canvas id="photoCanvas" width="600" height="400"></canvas>
        <div class="tools">
            <button onclick="addText()">Add Text</button>
            <button onclick="addFilter()">Apply Grayscale Filter</button>
            <div class="premium">
                <button onclick="addSepiaFilter()">Apply Sepia Filter</button>
                <button onclick="addBlurEffect()">Apply Blur Effect</button>
                <button onclick="addBrightnessEffect()">Adjust Brightness</button>
            </div>
            <button onclick="downloadPhoto()">Download Photo</button>
        </div>
        
        <h2>Video Editor</h2>
        <input type="file" id="videoUpload" accept="video/*">
        <video id="videoPlayer" controls></video>
        <div class="tools">
            <button onclick="trimVideo()">Trim Video (Basic)</button>
            <div class="premium">
                <button onclick="addVideoEffect()">Add Video Effect (Mock)</button>
            </div>
            <button onclick="downloadVideo()">Download Video</button>
        </div>
    </div>

    <script>
        let canvas = new fabric.Canvas('photoCanvas');
        let currentUser = null;
        let videoFile = null;
        let isSubscribed = false;

        // Mock sign-up/sign-in with local storage
        function signUp() {
            const name = document.getElementById('name').value;
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;
            if (!name || !email || !password) return alert('Fill all fields');
            localStorage.setItem(email, JSON.stringify({ name, password, subscribed: false }));
            document.getElementById('authMessage').textContent = 'Signed up! Now sign in.';
        }

        function signIn() {
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;
            const user = JSON.parse(localStorage.getItem(email));
            if (user && user.password === password) {
                currentUser = user;
                isSubscribed = user.subscribed || false;
                document.getElementById('auth').style.display = 'none';
                document.getElementById('editor').style.display = 'block';
                document.getElementById('userName').textContent = user.name;
                updatePremiumFeatures();
            } else {
                document.getElementById('authMessage').textContent = 'Invalid credentials.';
            }
        }

        function logout() {
            currentUser = null;
            isSubscribed = false;
            document.getElementById('auth').style.display = 'block';
            document.getElementById('editor').style.display = 'none';
        }

        // Subscription (mock payment)
        function subscribe(months) {
            if (!currentUser) return;
            const cost = months === 1 ? 110 : 250;
            if (confirm(`Subscribe for ${months} month(s) for ₹${cost}? (Mock payment - click OK to proceed)`)) {
                // In real app, integrate Razorpay or Stripe here
                isSubscribed = true;
                currentUser.subscribed = true;
                localStorage.setItem(currentUser.email, JSON.stringify(currentUser));
                document.getElementById('subMessage').textContent = `Subscribed for ${months} month(s)! Premium features unlocked.`;
                updatePremiumFeatures();
            }
        }

        function updatePremiumFeatures() {
            const premiums = document.querySelectorAll('.premium');
            premiums.forEach(el => el.style.display = isSubscribed ? 'block' : 'none');
        }

        // Photo editing
        document.getElementById('photoUpload').addEventListener('change', function(e) {
            const file = e.target.files[0];
            const reader = new FileReader();
            reader.onload = function(event) {
                fabric.Image.fromURL(event.target.result, function(img) {
                    canvas.clear();
                    canvas.add(img);
                    canvas.renderAll();
                });
            };
            reader.readAsDataURL(file);
        });

        function addText() {
            const text = new fabric.Text('Edit Me', { left: 100, top: 100 });
            canvas.add(text);
        }

        function addFilter() {
            const obj = canvas.getActiveObject();
            if (obj) obj.filters.push(new fabric.Image.filters.Grayscale());
            obj.applyFilters();
            canvas.renderAll();
        }

        // Premium filters/effects
        function addSepiaFilter() {
            if (!isSubscribed) return alert('Subscribe to unlock premium features!');
            const obj = canvas.getActiveObject();
            if (obj) obj.filters.push(new fabric.Image.filters.Sepia());
            obj.applyFilters();
            canvas.renderAll();
        }

        function addBlurEffect() {
            if (!isSubscribed) return alert('Subscribe to unlock premium features!');
            const obj = canvas.getActiveObject();
            if (obj) obj.filters.push(new fabric.Image.filters.Blur({ blur: 0.5 }));
            obj.applyFilters();
            canvas.renderAll();
        }

        function addBrightnessEffect() {
            if (!isSubscribed) return alert('Subscribe to unlock premium features!');
            const obj = canvas.getActiveObject();
            if (obj) obj.filters.push(new fabric.Image.filters.Brightness({ brightness: 0.2 }));
            obj.applyFilters();
            canvas.renderAll();
        }

        function downloadPhoto() {
            const link = document.createElement('a');
            link.download = 'edited-photo.png';
            link.href = canvas.toDataURL();
            link.click();
        }

        // Video editing (basic playback and mock trim)
        document.getElementById('videoUpload').addEventListener('change', function(e) {
            videoFile = e.target.files[0];
            const url = URL.createObjectURL(videoFile);
            document.getElementById('videoPlayer').src = url;
        });

        function trimVideo() {
            alert('Trimming not fully implemented in client-side. Use backend for real trimming.');
            // For demo, just pause at 5 seconds
            const video = document.getElementById('videoPlayer');
            video.currentTime = 5;
        }

        function addVideoEffect() {
            if (!isSubscribed) return alert('Subscribe to unlock premium features!');
            alert('Video effect applied (mock)!');
        }

        function downloadVideo() {
            if (videoFile) {
                const link = document.createElement('a');
                link.download = 'edited-video.mp4';
                link.href = URL.createObjectURL(videoFile);
                link.click();
            }
        }
    </script>
</body>
</html>
