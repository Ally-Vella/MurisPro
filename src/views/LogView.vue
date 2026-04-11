<template>
  <div class="main-content">
    <div class="section">
      <div class="header-with-button">
        <h2>系统操作日志</h2>
      </div>
      
      <div class="search-controls">
        <div class="filter-group">
          <label>操作用户:</label>
          <input type="text" v-model="filters.username" placeholder="输入用户名" @keyup.enter="fetchLogs">
        </div>

        <div class="filter-group">
          <label>目标ID:</label>
          <input type="text" v-model="filters.target_id" placeholder="如: M001 或 A-1" @keyup.enter="fetchLogs">
        </div>

        <div class="filter-group">
          <label>操作类型:</label>
          <select v-model="filters.action" @change="fetchLogs">
            <option value="">-- 全部操作 --</option>
            <option value="CREATE">新增 (CREATE)</option>
            <option value="UPDATE">修改 (UPDATE)</option>
            <option value="DELETE">删除 (DELETE)</option>
            <option value="MOVE">移笼 (MOVE)</option>
            <option value="BACKUP">备份 (BACKUP)</option>
          </select>
        </div>

        <div class="filter-group button-group">
          <button @click="fetchLogs" class="search-btn">
            <i class="material-icons">search</i> 搜索
          </button>
          <button @click="resetFilters" class="reset-btn">
            <i class="material-icons">refresh</i> 重置
          </button>
        </div>
      </div>
      
      <div v-if="loading" class="loading-overlay">
        <div class="loading-spinner"></div>
        <span>加载中...</span>
      </div>
      
      <table class="mouse-table">
        <thead>
          <tr>
            <th width="180">操作时间</th>
            <th width="120">操作人</th>
            <th width="120">动作</th>
            <th width="160">目标表(ID)</th>
            <th>详细描述</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="log in logs" :key="log.id">
            <td class="time-cell">{{ log.timestamp }}</td>
            <td class="user-cell"><i class="material-icons user-icon">person</i> {{ log.username }}</td>
            <td>
              <span :class="['action-badge', log.action.toLowerCase()]">{{ log.action }}</span>
            </td>
            <td><span class="target-badge">{{ log.target }}</span></td>
            <td class="detail-cell">{{ log.detail || '-' }}</td>
          </tr>
        </tbody>
      </table>
      
      <div v-if="logs.length === 0 && !loading" class="empty-state">
        <i class="material-icons">history_toggle_off</i>
        <p>没有找到符合条件的日志记录</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'

const api = axios.create({ baseURL: '/api' }) 

const logs = ref([])
const loading = ref(false)
const filters = reactive({
  username: '',
  target_id: '',
  action: ''
})

// 获取日志数据
const fetchLogs = async () => {
  loading.value = true
  try {
    const response = await api.get('/logs', {
      params: {
        username: filters.username,
        target_id: filters.target_id,
        action: filters.action,
        limit: 100 // 默认拉取前100条符合条件的数据
      }
    })
    logs.value = response.data
  } catch (error) {
    console.error('获取日志失败', error)
    toast.error('获取日志失败')
  } finally {
    loading.value = false
  }
}

// 重置过滤器
const resetFilters = () => {
  filters.username = ''
  filters.target_id = ''
  filters.action = ''
  fetchLogs()
}

onMounted(() => {
  fetchLogs()
})
</script>

<style scoped>
/* ---------------- 基础容器样式 (复用) ---------------- */
.section {
  margin-bottom: 30px;
  padding: 25px;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  position: relative;
  overflow: hidden;
}

.header-with-button {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  flex-wrap: wrap;
  gap: 15px;
}

.header-with-button h2 {
  font-size: 1.8rem;
  font-weight: 600;
  color: #2c3e50;
  margin: 0;
}

/* ---------------- 搜索控件样式 (复用) ---------------- */
.search-controls {
  display: flex;
  gap: 15px;
  margin-bottom: 25px;
  flex-wrap: wrap;
  align-items: flex-end;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.filter-group.button-group {
  margin-left: auto; /* 让按钮靠右 */
}

.filter-group label {
  font-weight: 500;
  white-space: nowrap;
  color: #4a5568;
}

.filter-group input,
.filter-group select {
  padding: 10px 15px;
  border: 1px solid #dcdfe6;
  border-radius: 6px;
  font-size: 0.95rem;
  min-width: 180px;
  transition: all 0.3s;
}

.filter-group input:focus,
.filter-group select:focus {
  border-color: #4a9bff;
  outline: none;
  box-shadow: 0 0 0 2px rgba(74, 155, 255, 0.2);
}

.search-btn, .reset-btn {
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 6px;
  font-weight: 500;
  transition: all 0.3s;
  border: 1px solid #dcdfe6;
}

.search-btn {
  background: #f5f7fa;
  color: #606266;
}

.search-btn:hover {
  background: #e4e7ed;
  color: #4a9bff;
}

.reset-btn {
  background: #f8f9fa;
  color: #606266;
}

.reset-btn:hover {
  background: #e2e8f0;
}

/* ---------------- 表格样式 (复用) ---------------- */
.mouse-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  border-radius: 8px;
  overflow: hidden;
}

.mouse-table th {
  background: #f8fafc;
  color: #64748b;
  font-weight: 600;
  text-align: left;
  padding: 14px 16px;
  border-bottom: 2px solid #e2e8f0;
}

.mouse-table td {
  padding: 14px 16px;
  border-bottom: 1px solid #f1f5f9;
  vertical-align: middle;
}

.mouse-table tbody tr:hover {
  background-color: #f8fafc;
}

/* ---------------- 日志专属定制样式 ---------------- */
.time-cell {
  color: #64748b;
  font-family: 'Courier New', Courier, monospace;
  font-size: 0.9rem;
}

.user-cell {
  font-weight: 600;
  color: #334155;
  display: flex;
  align-items: center;
  gap: 6px;
}

.user-icon {
  font-size: 18px;
  color: #94a3b8;
}

.detail-cell {
  color: #475569;
  line-height: 1.5;
}

/* 操作动作标签 */
.action-badge {
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  letter-spacing: 0.5px;
}

/* 柔和的标签颜色 */
.action-badge.create { background-color: #d1fae5; color: #059669; }
.action-badge.update { background-color: #e0f2fe; color: #0284c7; }
.action-badge.delete { background-color: #fee2e2; color: #dc2626; }
.action-badge.move   { background-color: #fef3c7; color: #d97706; }
.action-badge.backup { background-color: #f3e8ff; color: #9333ea; }

/* 目标表标签 */
.target-badge {
  background-color: #f1f5f9;
  color: #475569;
  padding: 4px 8px;
  border-radius: 4px;
  font-family: monospace;
  font-size: 0.85rem;
  border: 1px solid #e2e8f0;
}

/* ---------------- 加载与空状态 (复用) ---------------- */
.loading-overlay {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(255, 255, 255, 0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 10;
  border-radius: 12px;
}

.loading-spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left-color: #4a9bff;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin-bottom: 15px;
}

@keyframes spin { to { transform: rotate(360deg); } }

.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #94a3b8;
}

.empty-state .material-icons {
  font-size: 64px;
  color: #e2e8f0;
  margin-bottom: 15px;
}

.empty-state p {
  font-size: 1.1rem;
  margin: 0;
}

/* 响应式调整 */
@media (max-width: 900px) {
  .search-controls {
    flex-direction: column;
    align-items: stretch;
  }
  .filter-group.button-group {
    margin-left: 0;
    justify-content: flex-start;
  }
}
</style>