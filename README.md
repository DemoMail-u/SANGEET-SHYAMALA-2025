<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Offline Course Manager</title>
  <style>
    :root {
      --bg-color: #ffffff;
      --text-color: #000000;
    }
    body.dark {
      --bg-color: #1e1e1e;
      --text-color: #ffffff;
    }
    body {
      background-color: var(--bg-color);
      color: var(--text-color);
      font-family: sans-serif;
      padding: 20px;
      max-width: 1200px;
      margin: auto;
    }
    input, select, button {
      padding: 8px;
      margin: 5px 0;
      width: 100%;
      box-sizing: border-box;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: 15px;
    }
    .card {
      border: 1px solid #ccc;
      border-radius: 10px;
      padding: 15px;
      background-color: #f9f9f9;
      display: flex;
      flex-direction: column;
      gap: 5px;
    }
    .card.ongoing {
      border: 2px solid green;
      background-color: #e6ffe6;
    }
    .avatar img {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      object-fit: cover;
    }
    .top-controls {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 15px;
    }
  </style>
</head>
<body>

<h1>Offline Course Manager</h1>
<div class="top-controls">
  <input type="text" id="searchInput" placeholder="Search (e.g., dance, music)">
  <select id="themeSelector">
    <option value="light">Light Theme</option>
    <option value="dark">Dark Theme</option>
  </select>
</div>
<div id="courseList" class="grid"></div>

<h2>Add/Edit Course</h2>
<input type="text" id="name" placeholder="Course Name">
<input type="text" id="category" placeholder="Category (dance, music, etc.)">
<input type="text" id="day" placeholder="Day">
<input type="text" id="time" placeholder="Time (e.g., 17:00-18:00)">
<input type="text" id="fee" placeholder="Fee">
<input type="text" id="room" placeholder="Room">
<select id="type">
  <option value="group">Group</option>
  <option value="individual">Individual</option>
</select>
<input type="number" id="students" placeholder="Number of Students (if group)">
<input type="file" id="imageUpload" accept="image/*">
<button onclick="addCourse()">Add Course</button>

<script>
  const courses = JSON.parse(localStorage.getItem('courses') || '[]');

  function renderCourses() {
    const now = new Date();
    const currentHour = now.getHours();
    const search = document.getElementById('searchInput').value.toLowerCase();
    const list = document.getElementById('courseList');
    list.innerHTML = '';

    courses.filter(c => c.name.toLowerCase().includes(search) || c.category.toLowerCase().includes(search))
    .forEach((c, idx) => {
      const [start, end] = c.time.split('-').map(t => parseInt(t));
      const ongoing = now.getDay() === new Date(`1970-01-0${['sun','mon','tue','wed','thu','fri','sat'].indexOf(c.day.slice(0,3).toLowerCase())+1}`).getDay() && currentHour >= start && currentHour < end;

      const card = document.createElement('div');
      card.className = 'card' + (ongoing ? ' ongoing' : '');

      card.innerHTML = `
        <div class="avatar"><img src="${c.image}" alt="Teacher"></div>
        <input value="${c.name}" onchange="updateCourse(${idx}, 'name', this.value)">
        <input value="${c.category}" onchange="updateCourse(${idx}, 'category', this.value)">
        <input value="${c.day}" onchange="updateCourse(${idx}, 'day', this.value)">
        <input value="${c.time}" onchange="updateCourse(${idx}, 'time', this.value)">
        <input value="${c.fee}" onchange="updateCourse(${idx}, 'fee', this.value)">
        <input value="${c.room}" onchange="updateCourse(${idx}, 'room', this.value)">
        <select onchange="updateCourse(${idx}, 'type', this.value)">
          <option value="group" ${c.type === 'group' ? 'selected' : ''}>Group</option>
          <option value="individual" ${c.type === 'individual' ? 'selected' : ''}>Individual</option>
        </select>
        <input value="${c.students}" onchange="updateCourse(${idx}, 'students', this.value)" placeholder="Number of students">
        <input value="${c.image}" onchange="updateCourse(${idx}, 'image', this.value)">
        <button onclick="deleteCourse(${idx})">Delete</button>
      `;
      list.appendChild(card);
    });
  }

  function updateCourse(index, key, value) {
    courses[index][key] = value;
    saveCourses();
  }

  function deleteCourse(index) {
    courses.splice(index, 1);
    saveCourses();
  }

  function addCourse() {
    const name = document.getElementById('name').value;
    const category = document.getElementById('category').value;
    const day = document.getElementById('day').value;
    const time = document.getElementById('time').value;
    const fee = document.getElementById('fee').value;
    const room = document.getElementById('room').value;
    const type = document.getElementById('type').value;
    const students = document.getElementById('students').value;
    const imageFile = document.getElementById('imageUpload').files[0];

    if (!name || !category || !day || !time || !fee || !room || !type) return;

    if (imageFile) {
      const reader = new FileReader();
      reader.onload = function(e) {
        courses.push({ name, category, day, time, fee, room, type, students, image: e.target.result });
        clearInputs();
        saveCourses();
      };
      reader.readAsDataURL(imageFile);
    } else {
      courses.push({ name, category, day, time, fee, room, type, students, image: '' });
      clearInputs();
      saveCourses();
    }
  }

  function saveCourses() {
    localStorage.setItem('courses', JSON.stringify(courses));
    renderCourses();
  }

  function clearInputs() {
    document.querySelectorAll('input').forEach(i => i.value = '');
    document.getElementById('type').value = 'group';
  }

  document.getElementById('searchInput').addEventListener('input', renderCourses);
  document.getElementById('themeSelector').addEventListener('change', e => {
    document.body.className = e.target.value === 'dark' ? 'dark' : '';
  });

  window.onload = renderCourses;
</script>

</body>
</html>
