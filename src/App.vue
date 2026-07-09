<template>
  <div class="homepage">
    <div class="content">
      <div class="time-display">
        <span class="time">{{ currentTime }}</span>
        <span class="date">{{ currentDate }}</span>
      </div>
      <div class="search-box">
        <input type="text" v-model="searchQuery" placeholder="搜索..." @keyup.enter="handleSearch" />
        <button @click="handleSearch">搜索</button>
      </div>
      <div class="quick-links">
        <div
          v-for="(link, index) in quickLinks"
          :key="index"
          class="link-item"
          @click="openLink(link.url)"
          @contextmenu.prevent="openEditModal(index)"
        >
          <span class="link-icon">{{ link.icon }}</span>
          <span class="link-name">{{ link.name }}</span>
          <span class="link-edit-hint">右键编辑</span>
        </div>
        <div class="link-item add-link" @click="openAddModal">
          <span class="link-icon">+</span>
          <span class="link-name">添加</span>
        </div>
      </div>
    </div>

    <div v-if="showModal" class="modal-overlay" @click="closeModal">
      <div class="modal" @click.stop>
        <div class="modal-header">
          <span>{{ isEditing ? '编辑快捷链接' : '添加快捷链接' }}</span>
          <button class="close-btn" @click="closeModal">×</button>
        </div>
        <div class="modal-body">
          <div class="form-group">
            <label>名称</label>
            <input type="text" v-model="editForm.name" placeholder="链接名称" />
          </div>
          <div class="form-group">
            <label>网址</label>
            <input type="text" v-model="editForm.url" placeholder="https://..." />
          </div>
          <div class="form-group">
            <label>图标 (表情符号)</label>
            <input type="text" v-model="editForm.icon" placeholder="例如: 🌐" />
          </div>
        </div>
        <div class="modal-footer">
          <button v-if="isEditing" class="delete-btn" @click="deleteLink">删除</button>
          <button class="cancel-btn" @click="closeModal">取消</button>
          <button class="confirm-btn" @click="saveLink">保存</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const currentTime = ref('')
const currentDate = ref('')
const searchQuery = ref('')
const quickLinks = ref([])
const showModal = ref(false)
const isEditing = ref(false)
const editingIndex = ref(-1)
const editForm = ref({
  name: '',
  url: '',
  icon: '🌐'
})

const defaultLinks = [
  { name: 'Google', url: 'https://www.google.com', icon: '🌐' },
  { name: 'GitHub', url: 'https://github.com', icon: '💻' },
  { name: 'Stack Overflow', url: 'https://stackoverflow.com', icon: '❓' },
  { name: 'Vue', url: 'https://vuejs.org', icon: '💚' },
  { name: 'MDN', url: 'https://developer.mozilla.org', icon: '📚' },
  { name: 'YouTube', url: 'https://youtube.com', icon: '🎬' },
]

let timer = null

const loadLinks = () => {
  const saved = localStorage.getItem('quickLinks')
  if (saved) {
    try {
      quickLinks.value = JSON.parse(saved)
    } catch {
      quickLinks.value = [...defaultLinks]
    }
  } else {
    quickLinks.value = [...defaultLinks]
  }
}

const saveLinks = () => {
  localStorage.setItem('quickLinks', JSON.stringify(quickLinks.value))
}

const updateTime = () => {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  const seconds = String(now.getSeconds()).padStart(2, '0')
  currentTime.value = `${hours}:${minutes}:${seconds}`
  
  const year = now.getFullYear()
  const month = now.getMonth() + 1
  const day = now.getDate()
  const weekdays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六']
  const weekday = weekdays[now.getDay()]
  currentDate.value = `${year}年${month}月${day}日 ${weekday}`
}

const handleSearch = () => {
  if (searchQuery.value.trim()) {
    const url = `https://www.bing.com/search?q=${encodeURIComponent(searchQuery.value)}`
    window.open(url, '_blank')
    searchQuery.value = ''
  }
}

const openLink = (url) => {
  window.open(url, '_blank')
}

const openAddModal = () => {
  isEditing.value = false
  editingIndex.value = -1
  editForm.value = {
    name: '',
    url: '',
    icon: '🌐'
  }
  showModal.value = true
}

const openEditModal = (index) => {
  isEditing.value = true
  editingIndex.value = index
  const link = quickLinks.value[index]
  editForm.value = {
    name: link.name,
    url: link.url,
    icon: link.icon
  }
  showModal.value = true
}

const closeModal = () => {
  showModal.value = false
}

const saveLink = () => {
  if (!editForm.value.name.trim() || !editForm.value.url.trim()) {
    alert('请填写名称和网址')
    return
  }
  
  if (!editForm.value.url.startsWith('http')) {
    editForm.value.url = 'https://' + editForm.value.url
  }
  
  if (isEditing.value) {
    quickLinks.value[editingIndex.value] = { ...editForm.value }
  } else {
    quickLinks.value.push({ ...editForm.value })
  }
  
  saveLinks()
  closeModal()
}

const deleteLink = () => {
  if (confirm('确定要删除这个快捷链接吗？')) {
    quickLinks.value.splice(editingIndex.value, 1)
    saveLinks()
    closeModal()
  }
}

onMounted(() => {
  loadLinks()
  updateTime()
  timer = setInterval(updateTime, 1000)
})

onUnmounted(() => {
  if (timer) {
    clearInterval(timer)
  }
})
</script>

<style scoped>
.homepage {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  justify-content: center;
  align-items: center;
}

.content {
  text-align: center;
  color: white;
}

.time-display {
  margin-bottom: 40px;
}

.time {
  display: block;
  font-size: 8rem;
  font-weight: 300;
  letter-spacing: -4px;
}

.date {
  display: block;
  font-size: 1.5rem;
  opacity: 0.9;
  margin-top: 10px;
}

.search-box {
  display: flex;
  justify-content: center;
  margin-bottom: 50px;
}

.search-box input {
  width: 500px;
  height: 50px;
  padding: 0 20px;
  border: none;
  border-radius: 25px 0 0 25px;
  font-size: 1.1rem;
  outline: none;
}

.search-box button {
  height: 50px;
  padding: 0 30px;
  border: none;
  border-radius: 0 25px 25px 0;
  background: #4facfe;
  color: white;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.3s;
}

.search-box button:hover {
  background: #00f2fe;
}

.quick-links {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 20px;
}

.link-item {
  width: 100px;
  padding: 20px 15px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 15px;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  position: relative;
}

.link-item:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-5px);
}

.link-item:hover .link-edit-hint {
  opacity: 1;
}

.link-icon {
  font-size: 2rem;
}

.link-name {
  font-size: 0.9rem;
  opacity: 0.95;
}

.link-edit-hint {
  position: absolute;
  top: -30px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 0.7rem;
  opacity: 0;
  transition: opacity 0.3s;
  white-space: nowrap;
}

.add-link {
  background: rgba(255, 255, 255, 0.15);
  border: 2px dashed rgba(255, 255, 255, 0.4);
}

.add-link:hover {
  background: rgba(255, 255, 255, 0.25);
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background: white;
  border-radius: 15px;
  width: 400px;
  max-width: 90%;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.modal-header {
  padding: 20px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #333;
  font-weight: 600;
  font-size: 1.1rem;
}

.close-btn {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #999;
  transition: color 0.3s;
}

.close-btn:hover {
  color: #333;
}

.modal-body {
  padding: 20px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  color: #555;
  font-size: 0.9rem;
}

.form-group input {
  width: 100%;
  padding: 10px 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.3s;
}

.form-group input:focus {
  border-color: #667eea;
}

.modal-footer {
  padding: 20px;
  border-top: 1px solid #eee;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.modal-footer button {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  font-size: 0.95rem;
  cursor: pointer;
  transition: all 0.3s;
}

.cancel-btn {
  background: #f5f5f5;
  color: #666;
}

.cancel-btn:hover {
  background: #eee;
}

.confirm-btn {
  background: #667eea;
  color: white;
}

.confirm-btn:hover {
  background: #5a6fd6;
}

.delete-btn {
  background: #ff4757;
  color: white;
}

.delete-btn:hover {
  background: #e8384a;
}
</style>
