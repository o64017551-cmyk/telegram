<!doctype html>
<html lang="uz">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Yangı Dastur — Vazifalar Ro'yxati</title>
  <style>
    :root{font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;}
    body{margin:0;background:#f5f7fb;display:flex;align-items:center;justify-content:center;height:100vh;padding:20px}
    .app{width:100%;max-width:720px;background:#fff;border-radius:12px;box-shadow:0 8px 30px rgba(11,22,39,0.08);padding:24px}
    h1{margin:0 0 12px;font-size:20px}
    .controls{display:flex;gap:8px;margin-bottom:16px}
    input[type="text"]{flex:1;padding:10px;border:1px solid #e2e8f0;border-radius:8px;font-size:14px}
    button{padding:10px 14px;border-radius:8px;border:0;background:#2563eb;color:#fff;font-weight:600;cursor:pointer}
    button.secondary{background:#6b7280}
    ul{list-style:none;padding:0;margin:0;display:flex;flex-direction:column;gap:8px}
    li{display:flex;align-items:center;gap:10px;padding:10px;border-radius:8px;border:1px solid #edf2f7;background:#fcfdff}
    .task-text{flex:1;word-break:break-word}
    .task-text.completed{text-decoration:line-through;color:#6b7280}
    .small{font-size:13px;color:#475569}
    .actions{display:flex;gap:6px}
    .btn-icon{background:transparent;border:0;cursor:pointer;padding:6px;border-radius:6px}
    .btn-icon:hover{background:#f1f5f9}
    .empty{padding:18px;border-radius:8px;border:1px dashed #e2e8f0;text-align:center;color:#94a3b8}
    footer{margin-top:14px;display:flex;justify-content:space-between;align-items:center;color:#94a3b8;font-size:13px}
    .filters{display:flex;gap:6px}
    .filter-btn{padding:6px 8px;border-radius:6px;border:1px solid transparent;background:transparent;cursor:pointer}
    .filter-btn.active{border-color:#e2e8f0;background:#f8fafc}
  </style>
</head>
<body>
  <div class="app" role="application" aria-label="Vazifalar dasturi">
    <h1>Vazifalar — Yangı Dastur</h1>

    <div class="controls" aria-hidden="false">
      <input id="taskInput" type="text" placeholder="Yangi vazifa yozing... (Mas: Uy ishlarini tugatish)" />
      <button id="addBtn">Qoʻshish</button>
      <button id="clearBtn" class="secondary" title="Barcha tugatilganlarni o'chirish">Tozalash</button>
    </div>

    <ul id="taskList" aria-live="polite"></ul>

    <div id="empty" class="empty" hidden>Hozircha vazifa yoʻq. Yuqoridagi maydonga yozib qoʻshing.</div>

    <footer>
      <div class="small" id="count">0 vazifa</div>
      <div class="filters">
        <button class="filter-btn active" data-filter="all">Hammasi</button>
        <button class="filter-btn" data-filter="active">Faol</button>
        <button class="filter-btn" data-filter="completed">Tugatgan</button>
      </div>
    </footer>
  </div>

  <script>
    // Minimal, ammo to'liq funksionallik: qoʻshish, tahrirlash, o'chirish, belgilash, filtrlash, localStorage
    const input = document.getElementById('taskInput');
    const addBtn = document.getElementById('addBtn');
    const clearBtn = document.getElementById('clearBtn');
    const taskList = document.getElementById('taskList');
    const emptyEl = document.getElementById('empty');
    const countEl = document.getElementById('count');
    const filterBtns = document.querySelectorAll('.filter-btn');

    let tasks = JSON.parse(localStorage.getItem('tasks_v1') || '[]');
    let filter = 'all';

    function save() {
      localStorage.setItem('tasks_v1', JSON.stringify(tasks));
    }

    function uid() {
      return Date.now().toString(36) + Math.random().toString(36).slice(2,8);
    }

    function addTask(text) {
      if (!text || !text.trim()) return;
      tasks.unshift({ id: uid(), text: text.trim(), completed: false, createdAt: Date.now() });
      save();
      render();
      input.value = '';
      input.focus();
    }

    function toggleComplete(id) {
      tasks = tasks.map(t => t.id === id ? {...t, completed: !t.completed} : t);
      save();
      render();
    }

    function deleteTask(id) {
      tasks = tasks.filter(t => t.id !== id);
      save();
      render();
    }

    function editTask(id, newText) {
      tasks = tasks.map(t => t.id === id ? {...t, text: newText} : t);
      save();
      render();
    }

    function clearCompleted() {
      tasks = tasks.filter(t => !t.completed);
      save();
      render();
    }

    function filteredTasks() {
      if (filter === 'active') return tasks.filter(t => !t.completed);
      if (filter === 'completed') return tasks.filter(t => t.completed);
      return tasks;
    }

    function render() {
      const list = filteredTasks();
      taskList.innerHTML = '';
      if (list.length === 0) {
        emptyEl.hidden = false;
      } else {
        emptyEl.hidden = true;
        list.forEach(t => {
          const li = document.createElement('li');
          const checkbox = document.createElement('input');
          checkbox.type = 'checkbox';
          checkbox.checked = t.completed;
          checkbox.addEventListener('change', () => toggleComplete(t.id));

          const span = document.createElement('div');
          span.className = 'task-text' + (t.completed ? ' completed' : '');
          span.textContent = t.text;
          span.tabIndex = 0;
          span.title = 'Bosish bilan tahrirlash';

          // Double-click or Enter to edit
          span.addEventListener('dblclick', () => startEdit(t.id, span));
          span.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') startEdit(t.id, span);
          });

          const actions = document.createElement('div');
          actions.className = 'actions';

          const editBtn = document.createElement('button');
          editBtn.className = 'btn-icon';
          editBtn.title = 'Tahrirlash';
          editBtn.innerHTML = '✏️';
          editBtn.addEventListener('click', () => startEdit(t.id, span));

          const delBtn = document.createElement('button');
          delBtn.className = 'btn-icon';
          delBtn.title = 'Oʻchirish';
          delBtn.innerHTML = '🗑️';
          delBtn.addEventListener('click', () => {
            if (confirm('Vazifani oʻchiraysizmi?')) deleteTask(t.id);
          });

          li.appendChild(checkbox);
          li.appendChild(span);
          actions.appendChild(editBtn);
          actions.appendChild(delBtn);
          li.appendChild(actions);
          taskList.appendChild(li);
        });
      }
      const total = tasks.length;
      const remaining = tasks.filter(t => !t.completed).length;
      countEl.textContent = `${remaining} faol / ${total} jami`;
    }

    function startEdit(id, spanEl) {
      const task = tasks.find(t => t.id === id);
      if (!task) return;
      const inputEdit = document.createElement('input');
      inputEdit.type = 'text';
      inputEdit.value = task.text;
      inputEdit.style.width = '100%';
      inputEdit.addEventListener('keydown', (e) => {
        if (e.key === 'Enter') finishEdit();
        if (e.key === 'Escape') cancelEdit();
      });
      inputEdit.addEventListener('blur', finishEdit);
      spanEl.replaceWith(inputEdit);
      inputEdit.focus();

      function finishEdit() {
        const val = inputEdit.value.trim();
        if (!val) {
          if (confirm('Matn bo\'sh — vazifani oʻchirasizmi?')) deleteTask(id);
          else cancelEdit();
        } else {
          editTask(id, val);
        }
      }
      function cancelEdit() {
        render(); // qayta chizadi va inputni asl holatga qaytaradi
      }
    }

    // Event listeners
    addBtn.addEventListener('click', () => addTask(input.value));
    input.addEventListener('keydown', (e) => { if (e.key === 'Enter') addTask(input.value); });
    clearBtn.addEventListener('click', () => {
      if (confirm('Barcha tugatilgan vazifalarni o\'chirmoqchimisiz?')) clearCompleted();
    });

    filterBtns.forEach(btn => {
      btn.addEventListener('click', () => {
        filterBtns.forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        filter = btn.dataset.filter;
        render();
      });
    });

    // dastlabki render
    render();
  </script>
</body>
</html>
