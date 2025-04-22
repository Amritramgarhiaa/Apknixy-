<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>ViewStorm App Store</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f8f8f8; margin: 0; padding: 10px; }
    .search-bar { width: 100%; padding: 10px; font-size: 16px; margin-bottom: 10px; border-radius: 5px; border: 1px solid #ccc; }
    .category-section { margin-bottom: 30px; }
    .section-title { font-size: 18px; margin-bottom: 10px; font-weight: bold; color: #333; }
    .app-row { display: flex; overflow-x: auto; gap: 10px; padding-bottom: 10px; }
    .app-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
    .app-box { border: 1px solid #ccc; padding: 10px; background: white; border-radius: 8px; cursor: pointer; transition: 0.3s; }
    .app-box:hover { transform: scale(1.02); }
    .install-btn { background: #ff3c3c; color: white; padding: 8px; border: none; width: 100%; border-radius: 5px; font-weight: bold; margin-top: 5px; cursor: pointer; }
    .popup, .desc-popup { display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: white; padding: 20px; width: 90%; max-width: 400px; height: 80vh; overflow-y: auto; border-radius: 10px; z-index: 1000; box-shadow: 0 0 20px rgba(0,0,0,0.2); }
    .popup-header { display: flex; align-items: center; gap: 10px; }
    .popup-header img { width: 50px; height: 50px; border-radius: 8px; }
    .popup-info { color: #2196F3; font-size: 12px; margin-top: 5px; }
    .popup-screenshots { display: flex; overflow-x: auto; margin-top: 10px; gap: 10px; }
    .popup-screenshots img { height: 150px; border-radius: 8px; }
    .close-btn { background: gray; color: white; padding: 10px; border: none; width: 100%; border-radius: 5px; margin-top: 10px; cursor: pointer; }
    .desc-popup p { white-space: pre-wrap; }
    @media screen and (max-width: 600px) {
      .app-grid { grid-template-columns: repeat(2, 1fr); }
    }
  </style>
</head>
<body>
  <input type="text" class="search-bar" id="search" placeholder="Search apps..." onkeyup="searchApps()"/>
  <div id="categories-container"></div>
  <div id="search-results" class="category-section"></div>
  <div id="all-apps-container"></div>

  <!-- POPUPS -->
  <div id="popup" class="popup">
    <div class="popup-header">
      <img id="popup-logo" src="" alt="App Logo">
      <div>
        <h2 id="popup-title"></h2>
        <div id="popup-info" class="popup-info"></div>
      </div>
    </div>
    <p id="popup-meta"></p>
    <p id="popup-description"></p>
    <button class="install-btn" id="popup-install-btn">Install</button>
    <div class="popup-screenshots" id="popup-screenshots"></div>
    <button class="close-btn" onclick="closePopup()">Back</button>
  </div>

  <div id="desc-popup" class="desc-popup">
    <button class="close-btn" onclick="closeDescPopup()">Back</button>
    <p id="full-description"></p>
  </div>

  <script>
    let apps = [];
    const sectionNames = [
      "Trending", "Recommended for You", "Popular", "Top Rated",
      "Entertainment", "Productivity", "Music & Audio", "Shopping", 
      "Games", "Photography", "Education", "Tools", "Lifestyle"
    ];

    function fetchApps() {
      fetch("https://script.google.com/macros/s/AKfycbye3r1GVKSNvnyu0G1GnZZeRMFYyJPMPEolHH08VUdeCexkmNFyYL6JXOQnTBWQqltSFw/exec")
        .then(response => response.json())
        .then(data => {
          apps = data.sort(() => Math.random() - 0.5);
          displayRandomSections();
          displayAllAppsSection();
        })
        .catch(error => console.error("Error fetching data:", error));
    }

    function displayRandomSections() {
      const container = document.getElementById("categories-container");
      container.innerHTML = "";
      const appsCopy = [...apps];
      const shuffledSections = sectionNames.sort(() => 0.5 - Math.random());

      shuffledSections.forEach(section => {
        if (appsCopy.length === 0) return;
        const numApps = Math.floor(Math.random() * 6) + 3;
        const sectionApps = appsCopy.splice(0, numApps);
        if (sectionApps.length === 0) return;

        const sectionDiv = document.createElement("div");
        sectionDiv.className = "category-section";

        const title = document.createElement("div");
        title.className = "section-title";
        title.textContent = section;

        const row = document.createElement("div");
        row.className = "app-row";

        sectionApps.forEach(app => row.appendChild(createAppBox(app)));

        sectionDiv.appendChild(title);
        sectionDiv.appendChild(row);
        container.appendChild(sectionDiv);
      });
    }

    function displayAllAppsSection() {
      const allAppsContainer = document.getElementById("all-apps-container");
      allAppsContainer.innerHTML = "";

      const sectionDiv = document.createElement("div");
      sectionDiv.className = "category-section";

      const title = document.createElement("div");
      title.className = "section-title";
      title.textContent = "All Apps";

      const grid = document.createElement("div");
      grid.className = "app-grid";

      apps.forEach(app => grid.appendChild(createAppBox(app)));

      sectionDiv.appendChild(title);
      sectionDiv.appendChild(grid);
      allAppsContainer.appendChild(sectionDiv);
    }

    function createAppBox(app) {
      const box = document.createElement("div");
      box.className = "app-box";
      box.innerHTML = `
        <img src="${app.logo}" width="50" height="50"><br>
        <strong>${app.name}</strong><br>
        <span>${app.rating}★ | ${app.size}MB</span><br>
        <button class="install-btn">Install</button>
      `;
      box.onclick = () => showPopup(app);
      return box;
    }

    function showPopup(app) {
      document.getElementById("popup-logo").src = app.logo;
      document.getElementById("popup-title").textContent = app.name;

      let features = [];
      if (app.inAppPurchase?.toLowerCase() === "yes") features.push("• In-app purchases");
      if (app.containAds?.toLowerCase() === "yes") features.push("• Contains ads");
      document.getElementById("popup-info").textContent = features.join("  ");

      document.getElementById("popup-meta").innerHTML = `
        ${app.rating}★ | ${app.size}MB | ${app.downloads} Downloads | 
        ${app.madeForChildren?.toLowerCase() === "yes" ? "+3 age" : "+18 age"}
      `;

      const shortDesc = app.description.substring(0, 50) + "... ";
      const descElem = document.getElementById("popup-description");
      descElem.innerHTML = shortDesc;

      const more = document.createElement("span");
      more.textContent = "More+";
      more.style.color = "blue";
      more.style.cursor = "pointer";
      more.onclick = () => showFullDescription(app.description);
      descElem.appendChild(more);

      document.getElementById("popup-install-btn").onclick = () => window.location.href = app.apkLink;

      const screenshots = document.getElementById("popup-screenshots");
      screenshots.innerHTML = "";
      if (app.screenshots && app.screenshots.length > 0) {
        app.screenshots.forEach(url => {
          const img = document.createElement("img");
          img.src = url;
          screenshots.appendChild(img);
        });
      }

      document.getElementById("popup").style.display = "block";
    }

    function showFullDescription(desc) {
      document.getElementById("full-description").textContent = desc;
      document.getElementById("desc-popup").style.display = "block";
    }

    function closePopup() {
      document.getElementById("popup").style.display = "none";
    }

    function closeDescPopup() {
      document.getElementById("desc-popup").style.display = "none";
    }

    function searchApps() {
      const query = document.getElementById("search").value.toLowerCase();
      const resultSection = document.getElementById("search-results");
      resultSection.innerHTML = "";

      if (!query) {
        resultSection.style.display = "none";
        document.getElementById("categories-container").style.display = "block";
        document.getElementById("all-apps-container").style.display = "block";
        return;
      }

      const filteredApps = apps.filter(app =>
        app.name.toLowerCase().includes(query) ||
        app.description.toLowerCase().includes(query)
      );

      const sectionDiv = document.createElement("div");
      sectionDiv.className = "category-section";

      const title = document.createElement("div");
      title.className = "section-title";
      title.textContent = "Search Results";

      const row = document.createElement("div");
      row.className = "app-row";

      filteredApps.forEach(app => row.appendChild(createAppBox(app)));

      sectionDiv.appendChild(title);
      sectionDiv.appendChild(row);
      resultSection.appendChild(sectionDiv);

      resultSection.style.display = "block";
      document.getElementById("categories-container").style.display = "none";
      document.getElementById("all-apps-container").style.display = "none";
    }

    window.onload = fetchApps;
  </script>
</body>
</html>
