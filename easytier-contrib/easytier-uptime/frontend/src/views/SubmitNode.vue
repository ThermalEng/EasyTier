<template>
  <div class="submit-node">
    <!-- 页面头部 -->
    <div class="page-header">
      <el-button type="primary" @click="$router.back()" class="back-btn">
        <el-icon>
          <ArrowLeft />
        </el-icon>
        返回
      </el-button>
      <h1>提交共享节点</h1>
      <p class="subtitle">分享您的EasyTier节点，为社区贡献力量</p>
    </div>

    <el-row :gutter="20" justify="center">
      <el-col :span="16">
        <!-- 提交表单 -->
        <el-card class="form-card">
          <template #header>
            <div class="card-header">
              <el-icon>
                <Plus />
              </el-icon>
              <span>节点信息</span>
            </div>
          </template>
          <NodeForm ref="formRef" @submit="handleSubmit" :submitting="submitting" />
        </el-card>
      </el-col>

      <!-- 侧边栏信息 -->
      <el-col :span="8">
        <el-card class="info-card">
          <template #header>
            <div class="card-header">
              <el-icon>
                <InfoFilled />
              </el-icon>
              <span>提交须知</span>
            </div>
          </template>

          <div class="info-content">
            <div class="info-item">
              <el-icon color="#409EFF">
                <CircleCheck />
              </el-icon>
              <div>
                <h4>节点要求</h4>
                <p>确保您的节点稳定运行，具有良好的网络连接</p>
              </div>
            </div>

            <div class="info-item">
              <el-icon color="#67C23A">
                <Shield />
              </el-icon>
              <div>
                <h4>安全保障</h4>
                <p>我们会定期检查节点状态，确保服务质量</p>
              </div>
            </div>

            <div class="info-item">
              <el-icon color="#E6A23C">
                <Warning />
              </el-icon>
              <div>
                <h4>注意事项</h4>
                <p>请确保节点信息准确，避免提交虚假信息</p>
              </div>
            </div>

            <div class="info-item">
              <el-icon color="#F56C6C">
                <Delete />
              </el-icon>
              <div>
                <h4>移除条件</h4>
                <p>长期离线或不稳定的节点将被自动移除</p>
              </div>
            </div>
          </div>
        </el-card>

        <!-- 统计信息 -->
        <el-card class="stats-card">
          <template #header>
            <div class="card-header">
              <el-icon>
                <DataAnalysis />
              </el-icon>
              <span>社区统计</span>
            </div>
          </template>

          <div class="stats-content">
            <div class="stat-item">
              <div class="stat-number">{{ communityStats.totalNodes }}</div>
              <div class="stat-label">总节点数</div>
            </div>
            <div class="stat-item">
              <div class="stat-number">{{ communityStats.activeNodes }}</div>
              <div class="stat-label">在线节点</div>
            </div>
            <div class="stat-item">
              <div class="stat-number">{{ communityStats.totalConnections }}</div>
              <div class="stat-label">总连接数</div>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>


  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import { nodeApi } from '../api'
import {
  ArrowLeft,
  Plus,
  Monitor,
  Location,
  PriceTag,
  Link,
  Upload,
  RefreshLeft,
  InfoFilled,
  CircleCheck,
  Lock,
  Warning,
  Delete,
  DataAnalysis
} from '@element-plus/icons-vue'
import NodeForm from '../components/NodeForm.vue'

const formRef = ref()
const router = useRouter()
const submitting = ref(false)

// 社区统计数据
const communityStats = reactive({
  totalNodes: 0,
  activeNodes: 0,
  totalConnections: 0
})

const handleSubmit = async (submitData) => {
  try {
    const response = await nodeApi.createNode(submitData)

    if (response.success) {
      ElMessage.success('节点提交成功！')
      ElMessageBox.confirm(
        '节点已成功提交，是否返回节点列表查看？',
        '提交成功',
        {
          confirmButtonText: '查看列表',
          cancelButtonText: '继续提交',
          type: 'success'
        }
      ).then(() => {
        router.push('/')
      }).catch(() => {

      })
    } else {
      ElMessage.error(response.error || '提交失败，请重试')
    }
  } catch (error) {
    console.error('提交节点失败:', error)
    ElMessage.error('提交失败，请检查网络连接')
  } finally {
    submitting.value = false
  }
}

const fetchCommunityStats = async () => {
  try {
    const response = await nodeApi.getNodes({ page: 1, per_page: 1 })
    if (response.success && response.data) {
      communityStats.totalNodes = response.data.total

      // 获取活跃节点数
      const activeResponse = await nodeApi.getNodes({ page: 1, per_page: 1, is_active: true })
      if (activeResponse.success && activeResponse.data) {
        communityStats.activeNodes = activeResponse.data.total
      }

      // 获取所有节点来计算总连接数
      const allNodesResponse = await nodeApi.getNodes({ page: 1, per_page: 100 })
      if (allNodesResponse.success && allNodesResponse.data) {
        communityStats.totalConnections = allNodesResponse.data.items.reduce(
          (sum, node) => sum + node.current_connections, 0
        )
      }
    }
  } catch (error) {
    console.error('获取社区统计失败:', error)
  }
}

// 生命周期
onMounted(() => {
  fetchCommunityStats()
})
</script>

<style scoped>
.submit-node {
  padding: 20px;
  background-color: #f5f7fa;
  min-height: 100vh;
}

.page-header {
  text-align: center;
  margin-bottom: 30px;
  position: relative;
}

.back-btn {
  position: absolute;
  left: 0;
  top: 0;
}

.page-header h1 {
  color: #303133;
  margin-bottom: 10px;
  font-size: 28px;
  font-weight: 600;
}

.subtitle {
  color: #606266;
  font-size: 16px;
  margin: 0;
}


.info-card {
  margin-bottom: 20px;
}

.info-content {
  padding: 10px 0;
}

.info-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  margin-bottom: 20px;
}

.info-item:last-child {
  margin-bottom: 0;
}

.info-item h4 {
  margin: 0 0 5px 0;
  font-size: 14px;
  color: #303133;
}

.info-item p {
  margin: 0;
  font-size: 13px;
  color: #606266;
  line-height: 1.4;
}

.stats-card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.stats-card :deep(.el-card__header) {
  border-bottom-color: rgba(255, 255, 255, 0.2);
}

.stats-card :deep(.card-header) {
  color: white;
}

.stats-content {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.stat-item {
  text-align: center;
  padding: 15px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  backdrop-filter: blur(10px);
}

.stat-number {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 5px;
}

.stat-label {
  font-size: 12px;
  opacity: 0.8;
}

.terms-content {
  max-height: 400px;
  overflow-y: auto;
  padding: 10px;
}

.terms-content h3 {
  color: #303133;
  margin: 20px 0 10px 0;
  font-size: 16px;
}

.terms-content h3:first-child {
  margin-top: 0;
}

.terms-content p {
  margin: 5px 0;
  color: #606266;
  line-height: 1.6;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .submit-node {
    padding: 10px;
  }

  .page-header {
    margin-bottom: 20px;
  }

  .back-btn {
    position: static;
    margin-bottom: 10px;
  }

  .submit-section {
    flex-direction: column;
    align-items: center;
  }
}
</style>