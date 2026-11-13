
// ======================================
// MATURITNÝ ZOŠIT - JAVASCRIPT
// ======================================


// ===== 1. ULOŽENÉ TÉMY =====

const STORAGE_KEY = "maturitnyZositTopics";

let topics = [];

try {
  const savedTopics = JSON.parse(
    localStorage.getItem(STORAGE_KEY)
  );

  topics = Array.isArray(savedTopics)
    ? savedTopics
    : [];
} catch (error) {
  topics = [];
}

let editingId = null;
let activeSubject = null;
let currentView = "home";


// ===== 2. HTML ELEMENTY =====

const topicsContainer =
  document.getElementById("topicsContainer");

const topicModal =
  document.getElementById("topicModal");

const topicName =
  document.getElementById("topicName");

const topicSubject =
  document.getElementById("topicSubject");

const topicContent =
  document.getElementById("topicContent");

const searchInput =
  document.getElementById("searchInput");

const modalTitle =
  document.getElementById("modalTitle");

const saveTopicBtn =
  document.getElementById("saveTopicBtn");

const sectionTitle =
  document.getElementById("sectionTitle");


// ===== 3. UKLADANIE =====

function saveToStorage() {
  localStorage.setItem(
    STORAGE_KEY,
    JSON.stringify(topics)
  );
}


// ===== 4. OCHRANA TEXTU =====

function escapeHTML(text) {
  return String(text).replace(/[&<>"']/g, char => {
    const entities = {
      "&": "&amp;",
      "<": "&lt;",
      ">": "&gt;",
      '"': "&quot;",
      "'": "&#039;"
    };

    return entities[char];
  });
}


// ===== 5. ŠTATISTIKY =====

function updateStats() {
  const subjects = [
    ...new Set(
      topics.map(topic => topic.subject.trim())
    )
  ];

  document.getElementById("totalTopics").textContent =
    topics.length;

  document.getElementById("totalSubjects").textContent =
    subjects.length;
}


// ===== 6. ZOBRAZENIE TÉM =====

function renderTopics() {
  updateStats();

  const search = searchInput.value.toLowerCase();

  let filteredTopics = topics.filter(topic => {
    const matchesSearch =
      topic.name.toLowerCase().includes(search) ||
      topic.subject.toLowerCase().includes(search) ||
      topic.content.toLowerCase().includes(search);

    const matchesSubject =
      !activeSubject ||
      topic.subject === activeSubject;

    return matchesSearch && matchesSubject;
  });

  if (filteredTopics.length === 0) {
    topicsContainer.innerHTML = `
      <div class="empty-state">
        <div>📭</div>
        <h3>Žiadne témy</h3>
        <p>Zatiaľ tu nič nie je. Pridaj svoju prvú maturitnú tému!</p>
      </div>
    `;
    return;
  }

  topicsContainer.innerHTML = filteredTopics.map(topic => `
    <div class="topic-card">

      <div class="topic-top">
        <span class="topic-icon">📘</span>

        <span class="topic-subject">
          ${escapeHTML(topic.subject)}
        </span>
      </div>

      <h3>${escapeHTML(topic.name)}</h3>

      <p class="topic-preview">
        ${escapeHTML(topic.content || "Bez poznámok...").slice(0, 130)}
      </p>

      <div class="topic-actions">

        <button class="edit-btn" data-action="edit" data-id="${escapeHTML(topic.id)}">
          ✏️ Upraviť
        </button>

        <button class="delete-btn" data-action="delete" data-id="${escapeHTML(topic.id)}">
          🗑️
        </button>

      </div>

    </div>
  `).join("");
}


// ===== 7. OTVORENIE OKNA =====

function openModal(topic = null) {
  topicModal.style.display = "flex";

  if (topic) {
    editingId = topic.id;

    modalTitle.textContent = "Upraviť tému";
    saveTopicBtn.textContent = "Uložiť zmeny";

    topicName.value = topic.name;
    topicSubject.value = topic.subject;
    topicContent.value = topic.content;

  } else {
    editingId = null;

    modalTitle.textContent = "Nová maturitná téma";
    saveTopicBtn.textContent = "Uložiť tému";

    topicName.value = "";
    topicSubject.value = "";
    topicContent.value = "";
  }

  topicName.focus();
}


// ===== 8. ZATVORENIE OKNA =====

function closeModal() {
  topicModal.style.display = "none";
  editingId = null;
}

document.getElementById("cancelBtn")
  .addEventListener("click", closeModal);

topicModal.addEventListener("click", event => {
  if (event.target === topicModal) {
    closeModal();
  }
});


// ===== 9. PRIDANIE A ÚPRAVA TÉM =====

document.getElementById("addTopicBtn")
  .addEventListener("click", () => {
    openModal();
  });

saveTopicBtn.addEventListener("click", () => {

  const name = topicName.value.trim();
  const subject = topicSubject.value.trim();
  const content = topicContent.value.trim();

  if (!name || !subject) {
    alert("Vyplň názov témy a predmet!");
    return;
  }

  if (editingId) {

    const topic = topics.find(
      item => item.id === editingId
    );

    if (topic) {
      topic.name = name;
      topic.subject = subject;
      topic.content = content;
    }

  } else {

    topics.push({
      id: crypto.randomUUID
        ? crypto.randomUUID()
        : Date.now().toString() + Math.random(),

      name: name,
      subject: subject,
      content: content
    });

  }

  saveToStorage();
  closeModal();
  renderTopics();

});


// ===== 10. UPRAVOVANIE A MAZANIE =====

topicsContainer.addEventListener("click", event => {

  const button = event.target.closest("button");

  if (!button) return;

  const action = button.dataset.action;
  const id = button.dataset.id;

  const topic = topics.find(
    item => item.id === id
  );

  if (!topic) return;

  if (action === "edit") {
    openModal(topic);
  }

  if (action === "delete") {

    const confirmed = confirm(
      "Naozaj chceš vymazať túto tému?"
    );

    if (confirmed) {
      topics = topics.filter(
        item => item.id !== id
      );

      saveToStorage();
      renderTopics();
    }

  }

});


// ===== 11. VYHĽADÁVANIE =====

searchInput.addEventListener("input", () => {

  if (currentView === "subjects") {
    renderSubjects();
  } else {
    renderTopics();
  }

});


// ===== 12. PREDMETY =====

function renderSubjects() {

  updateStats();

  const subjects = [
    ...new Set(
      topics.map(topic => topic.subject.trim())
    )
  ];

  const search = searchInput.value.toLowerCase();

  const filteredSubjects = subjects.filter(subject =>
    subject.toLowerCase().includes(search)
  );

  sectionTitle.textContent = "Moje predmety";

  if (filteredSubjects.length === 0) {
    topicsContainer.innerHTML = `
      <div class="empty-state">
        <div>📚</div>
        <h3>Žiadne predmety</h3>
        <p>Pridaj tému a jej predmet sa tu automaticky zobrazí.</p>
      </div>
    `;
    return;
  }

  topicsContainer.innerHTML = filteredSubjects.map(subject => {

    const count = topics.filter(
      topic => topic.subject.trim() === subject
    ).length;

    return `
      <button class="subject-card"
        data-subject="${escapeHTML(subject)}">

        <span>📚</span>

        <h3>${escapeHTML(subject)}</h3>

        <p>${count} tém</p>

      </button>
    `;

  }).join("");

}


// ===== 13. KLIKNUTIE NA PREDMET =====

topicsContainer.addEventListener("click", event => {

  const subjectCard = event.target.closest(".subject-card");

  if (!subjectCard) return;

  activeSubject = subjectCard.dataset.subject;
  currentView = "home";

  document.getElementById("homeBtn")
    .classList.add("active");

  document.getElementById("subjectsBtn")
    .classList.remove("active");

  searchInput.value = "";

  sectionTitle.textContent = activeSubject;

  renderTopics();

});


// ===== 14. NAVIGÁCIA =====

document.getElementById("homeBtn")
  .addEventListener("click", () => {

    activeSubject = null;
    currentView = "home";

    searchInput.value = "";

    document.getElementById("homeBtn")
      .classList.add("active");

    document.getElementById("subjectsBtn")
      .classList.remove("active");

    sectionTitle.textContent = "Moje témy";

    renderTopics();

  });


document.getElementById("subjectsBtn")
  .addEventListener("click", () => {

    activeSubject = null;
    currentView = "subjects";

    searchInput.value = "";

    document.getElementById("subjectsBtn")
      .classList.add("active");

    document.getElementById("homeBtn")
      .classList.remove("active");

    renderSubjects();

  });


// ===== 15. SPUSTENIE STRÁNKY =====

renderTopics();
