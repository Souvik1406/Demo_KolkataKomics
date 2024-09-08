# Demo_KolkataKomics

To create a website like the design you've shown, with Firebase as the backend for storing and retrieving data, we can break it down into the following steps:

1. **Front-end (HTML, CSS, JavaScript):** This will create the structure of the webpage.
2. **Firebase Configuration:** Set up Firebase in your project to handle the back-end tasks like authentication and database operations.
3. **Firebase Database Setup:** Use Firestore or Realtime Database to store comics information.
4. **Firebase Authentication (optional):** If you need user login features.

### Step 1: HTML and CSS for the User Interface

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Comics App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f4f4f4;
        }
        .container {
            width: 60%;
            display: flex;
            justify-content: space-between;
        }
        .panel {
            border: 1px solid #ccc;
            border-radius: 10px;
            padding: 20px;
            width: 45%;
            background-color: #fff;
        }
        .panel h2 {
            text-align: center;
        }
        .comics-list, .publications-list {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
        }
        .tile {
            padding: 20px;
            background-color: #e3e3e3;
            text-align: center;
            border-radius: 5px;
            cursor: pointer;
        }
        .tile:hover {
            background-color: #ccc;
        }
    </style>
</head>
<body>

<div class="container">
    <div class="panel">
        <h2>Comics Languages</h2>
        <div id="searchBox">
            <input type="text" id="searchInput" placeholder="Search comics...">
        </div>
        <div class="comics-list" id="comicsList">
            <!-- Language tiles will go here -->
        </div>
    </div>

    <div class="panel">
        <h2>Comics Publications</h2>
        <div class="publications-list" id="publicationList">
            <!-- Publications will load here based on language -->
        </div>
    </div>
</div>

<script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-firestore.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-auth.js"></script>
<script src="app.js"></script>

</body>
</html>
```

### Step 2: Firebase Configuration and Setup (app.js)

- You will need to set up Firebase by creating a project at [Firebase Console](https://console.firebase.google.com/).
- Then, create a Firestore Database to store the comics categories (languages and publications).
- Add Firebase SDK credentials to your app.

```js
// app.js

// Firebase configuration (Replace with your credentials)
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_AUTH_DOMAIN",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_STORAGE_BUCKET",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID"
};

// Initialize Firebase
const app = firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// Example data for comics (replace with real-time fetching from Firebase)
const comicsData = {
    "Bengali": ["Kolkata Komics", "Indrajal Comics"],
    "Hindi": ["Amar Chitra Katha", "Diamond Comics"],
    "English": ["Marvel", "DC"],
    "Tamil": ["Lion Comics", "Muthu Comics"],
    "Telugu": ["Chandamama", "Chitti Comics"],
    "Malayalam": ["Balarama", "Poompatta"]
};

// Load languages
const comicsList = document.getElementById('comicsList');
const publicationList = document.getElementById('publicationList');

function loadLanguages() {
    comicsList.innerHTML = '';
    for (let language in comicsData) {
        const tile = document.createElement('div');
        tile.classList.add('tile');
        tile.textContent = `${language} Comics`;
        tile.onclick = () => loadPublications(language);
        comicsList.appendChild(tile);
    }
}

// Load publications based on language
function loadPublications(language) {
    publicationList.innerHTML = '';
    comicsData[language].forEach(pub => {
        const tile = document.createElement('div');
        tile.classList.add('tile');
        tile.textContent = pub;
        publicationList.appendChild(tile);
    });
}

// Search functionality
const searchInput = document.getElementById('searchInput');
searchInput.addEventListener('input', () => {
    const searchTerm = searchInput.value.toLowerCase();
    const filteredData = Object.keys(comicsData).filter(lang => lang.toLowerCase().includes(searchTerm));
    comicsList.innerHTML = '';
    filteredData.forEach(lang => {
        const tile = document.createElement('div');
        tile.classList.add('tile');
        tile.textContent = `${lang} Comics`;
        tile.onclick = () => loadPublications(lang);
        comicsList.appendChild(tile);
    });
});

// Initialize
loadLanguages();
```

### Step 3: Firestore Database Setup

In Firestore, create a collection named `comics` and store data in this format:

```
comics/
    {languageId}/
        name: "Bengali Comics"
        publications: ["Kolkata Komics", "Indrajal Comics"]
    {languageId}/
        name: "Hindi Comics"
        publications: ["Amar Chitra Katha", "Diamond Comics"]
    ...
```

To load this data dynamically from Firestore instead of using static data, update the code:

```js
function loadLanguagesFromFirebase() {
    db.collection("comics").get().then((querySnapshot) => {
        comicsList.innerHTML = '';
        querySnapshot.forEach((doc) => {
            const data = doc.data();
            const tile = document.createElement('div');
            tile.classList.add('tile');
            tile.textContent = `${data.name}`;
            tile.onclick = () => loadPublicationsFromFirebase(doc.id);
            comicsList.appendChild(tile);
        });
    });
}

function loadPublicationsFromFirebase(languageId) {
    db.collection("comics").doc(languageId).get().then((doc) => {
        const data = doc.data();
        publicationList.innerHTML = '';
        data.publications.forEach(pub => {
            const tile = document.createElement('div');
            tile.classList.add('tile');
            tile.textContent = pub;
            publicationList.appendChild(tile);
        });
    });
}

// Initialize with Firebase data
loadLanguagesFromFirebase();
```

### Step 4: Firebase Security Rules (Optional)

To restrict read and write access, configure your Firestore security rules in Firebase console:

```plaintext
service cloud.firestore {
  match /databases/{database}/documents {
    match /comics/{docId} {
      allow read: if true;  // Allow public read access
      allow write: if request.auth != null;  // Only authenticated users can write
    }
  }
}
```

### Conclusion:

This setup provides a basic structure for your comics app using Firebase as the backend. You can expand on it by adding more features like user authentication, advanced search, and more dynamic content.

To implement data filtering in your Firebase-backed comics app, you can enhance the search functionality and use Firestore queries to filter out the results based on user input.

Here’s how you can achieve data filtering, both for **languages** and **publications**, using Firebase Firestore:

### Step 1: Update Firestore Database Structure

Ensure your Firestore database is structured in a way that allows efficient querying. Here’s how you can structure it:

```
comics/
    {languageId}/
        name: "Bengali Comics"
        publications: ["Kolkata Komics", "Indrajal Comics"]
    {languageId}/
        name: "Hindi Comics"
        publications: ["Amar Chitra Katha", "Diamond Comics"]
    ...
```

### Step 2: Modify the Search and Filter Functionality

We will update the search functionality to filter data directly from Firestore. The goal is to allow the user to search for a language or publication, and show only the matching results.

#### Updated `app.js` Code with Filtering

```js
// Firebase configuration (Replace with your credentials)
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_AUTH_DOMAIN",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_STORAGE_BUCKET",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID"
};

// Initialize Firebase
const app = firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

// HTML Elements
const comicsList = document.getElementById('comicsList');
const publicationList = document.getElementById('publicationList');
const searchInput = document.getElementById('searchInput');

// Load languages from Firebase with filtering option
function loadLanguagesFromFirebase(filter = '') {
    comicsList.innerHTML = '';
    let query = db.collection("comics");

    if (filter) {
        // Search for languages that contain the filter term
        query = query.where('name', '>=', filter).where('name', '<=', filter + '\uf8ff');
    }

    query.get().then((querySnapshot) => {
        querySnapshot.forEach((doc) => {
            const data = doc.data();
            const tile = document.createElement('div');
            tile.classList.add('tile');
            tile.textContent = `${data.name}`;
            tile.onclick = () => loadPublicationsFromFirebase(doc.id);
            comicsList.appendChild(tile);
        });

        if (comicsList.innerHTML === '') {
            comicsList.innerHTML = '<p>No comics found</p>';
        }
    });
}

// Load publications based on selected language with optional filter
function loadPublicationsFromFirebase(languageId, filter = '') {
    publicationList.innerHTML = '';

    db.collection("comics").doc(languageId).get().then((doc) => {
        const data = doc.data();
        let publications = data.publications;

        if (filter) {
            // Filter publications by name
            publications = publications.filter(pub => pub.toLowerCase().includes(filter.toLowerCase()));
        }

        publications.forEach(pub => {
            const tile = document.createElement('div');
            tile.classList.add('tile');
            tile.textContent = pub;
            publicationList.appendChild(tile);
        });

        if (publicationList.innerHTML === '') {
            publicationList.innerHTML = '<p>No publications found</p>';
        }
    });
}

// Search functionality to filter both languages and publications
searchInput.addEventListener('input', () => {
    const searchTerm = searchInput.value.toLowerCase();

    if (searchTerm) {
        // Filter languages based on search term
        loadLanguagesFromFirebase(searchTerm);
    } else {
        // Load all languages if no search term is entered
        loadLanguagesFromFirebase();
    }
});

// Initialize the app by loading all languages
loadLanguagesFromFirebase();
```

### Key Features in the Code:

1. **Language Filtering:**
   - The search box filters the languages displayed based on the user's input. 
   - It uses a Firestore query with `.where()` to get the matching results, using the search term to compare the `name` field of the `comics` collection.

2. **Publication Filtering:**
   - When a language is selected, it shows all the publications associated with that language.
   - If the user provides a search term, the publications are filtered locally by checking if the publication name contains the search term.

### Step 3: Explanation of the Filter Logic

- **Firestore Query (`where`)**: In Firebase Firestore, the `.where()` method filters the documents based on conditions. Here, we use `>=` and `<=` conditions to filter results based on the `name` field. This is a basic string search where the input is matched against the `name` values stored in Firestore.

- **Client-side Filtering for Publications**: Since the publications are stored as arrays, we filter them on the client side. After fetching the list of publications for a selected language, we apply the `Array.filter()` method to match publications with the search term.

- **Handling Empty Results**: If no matching comics or publications are found, a message is displayed indicating that there are "No comics found" or "No publications found."

### Step 4: Firebase Security Rules (Optional)

If you want to ensure that only authenticated users can modify the comics data, you can update your Firebase security rules as mentioned earlier. For a public read-only app, you can set:

```plaintext
service cloud.firestore {
  match /databases/{database}/documents {
    match /comics/{docId} {
      allow read: if true;  // Public read access
      allow write: if request.auth != null;  // Only authenticated users can write
    }
  }
}
```

### Step 5: Deploy the Website

Once you’ve built the app, you can host it on Firebase Hosting by following these steps:
1. Run `firebase init` and select Hosting.
2. Choose the folder that contains your HTML, CSS, and JavaScript files.
3. Deploy your app using `firebase deploy`.

This will give you a live, Firebase-backed web application with filtering capabilities for the comic languages and publications.

Let me know if you'd like further details on Firebase setup or assistance with deploying!
