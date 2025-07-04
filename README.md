<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GroveSqueeze Order Messaging Tool</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }

        .header {
            text-align: center;
            margin-bottom: 30px;
            padding: 20px;
            background: linear-gradient(135deg, #2c5530, #4a7c59);
            color: white;
            border-radius: 10px;
        }

        .controls {
            margin-bottom: 20px;
            padding: 15px;
            background: white;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            gap: 10px;
            align-items: center;
        }

        .btn {
            background: #4a7c59;
            color: white;
            border: none;
            padding: 12px 18px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 500;
            transition: all 0.2s;
            width: 300px;
            text-align: left;
        }

        .btn:hover {
            background: #2c5530;
            transform: translateY(-1px);
        }

        .btn:active {
            transform: translateY(0);
        }

        .btn-custom {
            background: #4a7c59;
            text-align: center;
        }

        .btn-custom:hover {
            background: #2c5530;
        }

        .btn-danger {
            background: #dc3545;
            text-align: center;
        }

        .btn-danger:hover {
            background: #c82333;
        }

        .form-control {
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 6px;
            font-size: 14px;
            width: 278px; /* 300px minus padding */
            height: 80px;
            resize: none;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        .form-control:focus {
            outline: none;
            border-color: #4a7c59;
        }

        .preview {
            margin: 15px 0;
            padding: 15px;
            background: white;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            font-size: 14px;
            color: #333;
            width: 300px;
            text-align: center;
        }

        .alert {
            padding: 15px;
            margin-bottom: 20px;
            border-radius: 8px;
            font-weight: 500;
            width: 300px;
            text-align: center;
        }

        .alert-success {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }

        .alert-error {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }

        .manual-copy-container {
            margin: 10px 0;
            width: 300px;
        }

        .manual-copy-textarea {
            width: 100%;
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 6px;
            font-size: 14px;
            resize: none;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        .manual-copy-textarea:focus {
            outline: none;
            border-color: #4a7c59;
        }

        .google-voice-btn {
            background: #007bff;
            color: white;
            border: none;
            padding: 10px 16px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 500;
            transition: all 0.2s;
            width: 300px;
            text-align: center;
            margin-top: 5px;
        }

        .google-voice-btn:hover {
            background: #0056b3;
            transform: translateY(-1px);
        }

        .google-voice-btn:active {
            transform: translateY(0);
        }

        @media (max-width: 340px) {
            .btn, .form-control, .preview, .alert, .manual-copy-container, .google-voice-btn {
                width: 260px;
            }
            .form-control, .manual-copy-textarea {
                width: 238px; /* Adjust for padding */
            }
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>🍊 GroveSqueeze Order Messaging Tool</h1>
        <p>Send quick order updates to customers via Google Voice</p>
    </div>

    <div id="alertContainer"></div>

    <div class="controls">
        <div class="preview" id="preview">Click a message or type a custom message to preview it here.</div>
        <button class="btn" onclick="handleMessage('Your order is ready for pickup! Please visit us at your convenience.')">Order is Ready</button>
        <button class="btn" onclick="handleMessage('We apologize, but your order is delayed. We\'ll notify you once it\'s ready.')">Order is Delayed</button>
        <button class="btn" onclick="handleMessage('Please return to the window regarding your order.')">Return to Window</button>
        <button class="btn" onclick="handleMessage('Thank you for your order! It’s being prepared and will be ready soon.')">Order in Progress</button>
        <button class="btn" onclick="handleMessage('We’re sorry, but there’s an issue with your order. Please contact us.')">Issue with Order</button>
        <button class="btn" onclick="handleMessage('Your order has been canceled. Let us know if you have any questions.')">Order Canceled</button>
        <button class="btn" onclick="handleMessage('Your order is out for delivery and will arrive soon.')">Order Out for Delivery</button>
        <textarea id="customMessage" class="form-control" placeholder="Type your custom message here" oninput="previewCustomMessage()"></textarea>
        <button id="sendCustomButton" class="btn btn-custom" onclick="sendCustomMessage()">Send Custom Message</button>
        <button id="clearCustomButton" class="btn btn-danger" onclick="clearCustomMessage()">Clear Custom Message</button>
    </div>

    <script>
        function handleMessage(message) {
            const preview = document.getElementById('preview');
            preview.textContent = message;

            // Check if running in an iframe
            const isInIframe = window.self !== window.top;
            if (isInIframe) {
                showAlert('Clipboard access may be restricted in this environment. Select the message below to copy manually or open in Google Voice.', 'error');
                showManualCopyTextarea(message);
                return;
            }

            // Try Clipboard API if available and in a secure context
            if (navigator.clipboard && navigator.clipboard.writeText && window.isSecureContext) {
                navigator.clipboard.writeText(message).then(() => {
                    showAlert('Message copied to clipboard for Google Voice.', 'success');
                }).catch((error) => {
                    console.error('Clipboard error:', error);
                    showAlert('Unable to copy to clipboard. Select the message below to copy manually or open in Google Voice.', 'error');
                    showManualCopyTextarea(message);
                });
            } else {
                showAlert('Clipboard access not available. Select the message below to copy manually or open in Google Voice.', 'error');
                showManualCopyTextarea(message);
            }
        }

        function showManualCopyTextarea(message) {
            const controls = document.querySelector('.controls');
            const existingContainer = document.querySelector('.manual-copy-container');
            if (existingContainer) existingContainer.remove(); // Remove previous textarea

            const container = document.createElement('div');
            container.className = 'manual-copy-container';

            const textarea = document.createElement('textarea');
            textarea.className = 'manual-copy-textarea';
            textarea.value = message;
            textarea.readOnly = true;
            textarea.rows = 3;
            textarea.onclick = () => textarea.select();
            container.appendChild(textarea);

            const googleVoiceBtn = document.createElement('button');
            googleVoiceBtn.className = 'google-voice-btn';
            googleVoiceBtn.textContent = 'Open in Google Voice';
            googleVoiceBtn.onclick = () => {
                window.open(`https://voice.google.com/messages?text=${encodeURIComponent(message)}`, '_blank');
                showAlert('Opening Google Voice. Paste the message if needed.', 'success');
            };
            container.appendChild(googleVoiceBtn);

            controls.appendChild(container);
            textarea.select();
            showAlert('Please press Ctrl+C (or Cmd+C) to copy the message, or use the Google Voice button.', 'error', 10000);
        }

        function previewCustomMessage() {
            const customMessage = document.getElementById('customMessage').value;
            const preview = document.getElementById('preview');
            preview.textContent = customMessage || 'Click a message or type a custom message to preview it here.';
        }

        function sendCustomMessage() {
            const customMessage = document.getElementById('customMessage').value;
            if (!customMessage) {
                showAlert('Please type a custom message before sending.', 'error');
                return;
            }
            handleMessage(customMessage);
        }

        function clearCustomMessage() {
            const customMessage = document.getElementById('customMessage');
            const preview = document.getElementById('preview');
            const manualCopyContainer = document.querySelector('.manual-copy-container');
            if (manualCopyContainer) manualCopyContainer.remove();
            customMessage.value = '';
            preview.textContent = 'Click a message or type a custom message to preview it here.';
            showAlert('Custom message cleared.', 'success');
        }

        function showAlert(message, type = 'success', duration = 5000) {
            const alertContainer = document.getElementById('alertContainer');
            const alertDiv = document.createElement('div');
            alertDiv.className = `alert alert-${type}`;
            alertDiv.textContent = message;
            alertContainer.appendChild(alertDiv);

            setTimeout(() => {
                alertDiv.remove();
            }, duration);
        }
    </script>
</body>
</html>
