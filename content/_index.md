---
title: premy - A drawing app for anyone.
---

<script type="module">
  import "https://cdn.jsdelivr.net/npm/premy@10.0.2";
</script>

<style>
  .intro {
    transform: unset;
  }

  .navbar-fixed-bottom, .navbar-fixed-top {
    position: absolute;
  }

  #action-container {
    display: flex;
    align-items: center;
    flex-direction: column;
    gap: 16px;
    margin-bottom: 10px;
  }

  #open-button {
    all: unset;
    background: #000000;
    border-radius: 4px;
    color: #ffffff;
    cursor: pointer;
    font-size: x-large;
    padding: 2px 8px;
  }
  #open-button:focus {
    outline: 2px solid #000000;
  }
  #open-button:hover {
    background: #333333;
  }

  #install-button {
    display: none;
    color: unset;
    text-decoration: unset;
  }
  @media (display-mode: browser) {
    #install-button {
      display: unset;
    }
  }
  #install-button button {
    all: unset;
    border: 1px solid #000000;
    border-radius: 4px;
    cursor: pointer;
    padding: 2px 8px;
  }
  #install-button button:focus {
    outline: 2px solid #000000;
  }
  #install-button button:hover {
    background: #f5f5f5;
  }

  #examples-container {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 8px;
  }
</style>

&nbsp;

<div id="action-container">
  <a id="install-button" href="https://help.hata6502.com/premy%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B-61818b0489e586002278f64c" target="_blank"><button>Install app</button></a>
  <button id="open-button">Open canvas</button>
  <a href="https://twitter.com/premy_draw?ref_src=twsrc%5Etfw" class="twitter-follow-button" data-show-count="false">Follow @premy_draw</a>
</div>

&nbsp;

<div id="examples-container"></div>

<premy-dialog id="dialog"></premy-dialog>

<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<script type="module">
  if ("serviceWorker" in navigator) {
    await navigator.serviceWorker.register("./serviceWorker.js");
  }
</script>

<script type="module">
  const dialog = document.querySelector("#dialog");
  const openButton = document.querySelector("#open-button");
  dialog.addEventListener("premyClose", () => {
    dialog.removeAttribute("open");
  });

  const premyDBOpenRequest = indexedDB.open("premy", 2);

  premyDBOpenRequest.onsuccess = () => {
    const premyDB = premyDBOpenRequest.result;

    openButton.addEventListener("click", async () => {
      const transaction = premyDB.transaction(["history"], "readonly");
      const historyStore = transaction.objectStore("history");
      const historyGetAllRequest = historyStore.getAll();
      await new Promise((resolve, reject) => {
        historyGetAllRequest.onsuccess = resolve;
        historyGetAllRequest.onerror = reject;
      });
      const history = historyGetAllRequest.result;
      dialog.setHistory(history);
      dialog.setAttribute("open", "");
    });

    dialog.addEventListener("premyHistoryChange", async (event) => {
      const { historyMaxLength, pushed } = event.detail;

      const transaction = premyDB.transaction(["history"], "readwrite");
      const historyStore = transaction.objectStore("history");
      for (const dataURL of pushed) {
        historyStore.add(dataURL);
      }

      const historyGetAllKeysRequest = historyStore.getAllKeys();
      await new Promise((resolve, reject) => {
        historyGetAllKeysRequest.onsuccess = resolve;
        historyGetAllKeysRequest.onerror = reject;
      });
      const historyKeys = historyGetAllKeysRequest.result;
      const removedHistoryKeys = historyKeys.slice(
        0,
        Math.max(historyKeys.length - historyMaxLength, 0)
      );
      for (const historyKey of removedHistoryKeys) {
        historyStore.delete(historyKey);
      }
    });
  };

  premyDBOpenRequest.onerror = () => {
    openButton.addEventListener("click", () => {
      dialog.setAttribute("open", "");
    });
  };

  premyDBOpenRequest.onupgradeneeded = async (event) => {
    const premyDB = premyDBOpenRequest.result;
    const transaction = premyDBOpenRequest.transaction;
    let version = event.oldVersion;

    if (version === 0) {
      const etcStore = premyDB.createObjectStore("etc");
      const image = localStorage.getItem("premy-image");
      if (image) {
        etcStore.put([image], "history");
      }
      localStorage.removeItem("premy-image");
      version++;
    }

    if (version === 1) {
      const etcStore = transaction.objectStore("etc");
      const historyGetRequest = etcStore.get("history");
      await new Promise((resolve, reject) => {
        historyGetRequest.onsuccess = resolve;
        historyGetRequest.onerror = reject;
      });
      const history = historyGetRequest.result ?? [];

      const historyStore = premyDB.createObjectStore("history", {
        autoIncrement: true,
      });
      for (const dataURL of history) {
        historyStore.add(dataURL);
      }

      premyDB.deleteObjectStore("etc");
      version++;
    }
  };
</script>

<script type="module">
  const examplesResponse = await fetch("./examples.json");
  if (!examplesResponse.ok) {
    throw new Error("Failed to fetch examples.json");
  }
  const examples = await examplesResponse.json();

  const links =
    [...examples.relatedPages.links1hop]
    .sort(() => Math.random() - 0.5);
  const examplesContainerElement = document.querySelector("#examples-container");

  for (const { title, image } of links) {
    const linkElement = document.createElement("a");
    linkElement.href = `https://scrapbox.io/hata6502/${encodeURIComponent(title)}`;
    linkElement.target = "_blank";

    const imageElement = document.createElement("img");
    imageElement.alt = title;
    imageElement.src = image;
    imageElement.loading = "lazy";
    imageElement.style.width = "224px";
    imageElement.style.borderBottom = "1px solid #337ab7";

    linkElement.append(imageElement);
    examplesContainerElement.append(linkElement);
  }
</script>
