const STORAGE_KEY = 'complaint-box-submissions';

const form = document.getElementById('complaintForm');
const complaintList = document.getElementById('complaintList');

function getSubmissions() {
  const raw = localStorage.getItem(STORAGE_KEY);
  try {
    return raw ? JSON.parse(raw) : [];
  } catch (error) {
    console.error('Failed to parse saved complaints:', error);
    return [];
  }
}

function saveSubmissions(submissions) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(submissions));
}

function renderComplaints() {
  const submissions = getSubmissions();

  if (!submissions.length) {
    complaintList.innerHTML = '<p class="empty-state">No complaints submitted yet.</p>';
    return;
  }

  complaintList.innerHTML = submissions
    .slice()
    .reverse()
    .map((item) => {
      const priorityClass = (item.priority || 'Low').toLowerCase();
      return `
        <article class="complaint-item">
          <div class="meta">
            <h3>${escapeHtml(item.name)}</h3>
            <span class="badge ${priorityClass}">${escapeHtml(item.priority)}</span>
          </div>
          <p><strong>${escapeHtml(item.category)}</strong> · ${escapeHtml(item.date)}</p>
          <p>${escapeHtml(item.message)}</p>
        </article>
      `;
    })
    .join('');
}

function escapeHtml(value) {
  return String(value)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

form.addEventListener('submit', (event) => {
  event.preventDefault();

  const formData = new FormData(form);
  const newComplaint = {
    name: formData.get('name'),
    email: formData.get('email'),
    category: formData.get('category'),
    priority: formData.get('priority'),
    message: formData.get('message'),
    date: new Date().toLocaleString()
  };

  const submissions = getSubmissions();
  submissions.push(newComplaint);
  saveSubmissions(submissions);
  renderComplaints();
  form.reset();
});

renderComplaints();
