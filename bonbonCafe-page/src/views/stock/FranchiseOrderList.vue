<template>
  <v-container class="py-4 hei" fluid>
    <v-row dense>
      <v-col cols="12" md="10" offset-md="1">
        <v-card class="pa-6 elevation-2" style="min-height: 650px;">
          <v-typography class="list" align="center">재고 관리 /</v-typography>
          <v-typography class="title" align="center">가맹점 재고 신청 내역</v-typography>

          <br /><br />

          <!-- 🔍 검색 + 상태 필터 -->
          <v-row class="mb-4" align="center" justify="space-between" style="gap: 12px;">
            <v-col cols="12" md="6">
              <v-text-field v-model="search" label="가맹점 이름 검색" prepend-inner-icon="mdi-magnify" variant="outlined"
                density="comfortable" hide-details @keyup.enter="fetchHistories" />
            </v-col>

            <v-col cols="12" md="4">
              <v-select v-model="selectedStatus" :items="statusOptions" label="신청 처리 현황" item-title="label"
                item-value="value" clearable density="comfortable" variant="outlined" hide-details
                @update:model-value="onStatusChange" />
            </v-col>
          </v-row>

          <!-- 📋 테이블 -->
          <v-data-table :headers="headers" :items="filteredHistories" class="rounded-table" density="comfortable"
            hide-default-footer>
            <template #item="{ item, index }">
              <tr @click="goToDetailPage(item)" style="cursor: pointer;">
                <td class="text-center">{{ totalElements - (page - 1) * pageSize - index }}</td>
                <td class="text-center">{{ item.franchiseName }}</td>
                <td class="text-center">{{ item.ingredientName }}</td>
                <td class="text-center">{{ item.quantity }} {{ item.unit || '' }}</td>
                <td class="text-center">{{ formatDate(item.date) }}</td>
                <td class="text-center">{{ statusLabel(item.historyStatus) }}</td>
              </tr>
            </template>
          </v-data-table>

          <!-- 📄 페이지네이션 -->
          <v-row class="mt-4 justify-end">
            <v-pagination v-model="page" :length="totalPages" :total-visible="5" color="primary" @input="fetchStocks" />
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'
import apiClient from '@/api'

const router = useRouter()

const histories = ref([])
const loading = ref(false)
const search = ref('')
const selectedStatus = ref(null)

const page = ref(1)
const pageSize = 10
const totalPages = ref(1)
const totalElements = ref(0)

const statusOptions = [
  { value: 'REQUESTED', label: '신청 완료' },
  { value: 'APPROVED', label: '승인 완료' },
  { value: 'REJECTED', label: '승인 거부' },
  { value: 'SHIPPED', label: '배송 진행 중' },
  { value: 'DELIVERED', label: '배송 완료' }
]

const headers = [
  { title: '번호', key: 'number', align: 'center', sortable: false },
  { title: '가맹점명', key: 'franchiseName', align: 'center', sortable: false },
  { title: '재료명', key: 'ingredientName', align: 'center', sortable: false },
  { title: '수량', key: 'quantity', align: 'center', sortable: false },
  { title: '신청일', key: 'date', align: 'center', sortable: false },
  { title: '상태', key: 'historyStatus', align: 'center', sortable: false }
]

const fetchHistories = async () => {
  loading.value = true
  try {
    const { data } = await apiClient.get('/franchiseOrder/headquarter/franchise-history-list', {
      params: {
        page: page.value - 1,
        status: selectedStatus.value || null
      }
    })
    histories.value = data.content || []
    totalPages.value = data.totalPages
    totalElements.value = data.totalElements
  } catch (e) {
    console.error('❌ 전체 조회 실패', e)
    alert('조회 실패')
  } finally {
    loading.value = false
  }
}

const onStatusChange = () => {
  page.value = 1
  fetchHistories()
}

const filteredHistories = computed(() => {
  if (!search.value.trim()) return histories.value
  return histories.value.filter(h =>
    h.franchiseName?.toLowerCase().includes(search.value.toLowerCase())
  )
})

const formatDate = (dateStr) => {
  return new Date(dateStr).toLocaleDateString()
}

const statusLabel = (status) => {
  const map = {
    REQUESTED: '신청 완료',
    APPROVED: '승인 완료',
    REJECTED: '승인 거부',
    SHIPPED: '배송 진행 중',
    DELIVERED: '배송 완료'
  }
  return map[status] || status
}

const goToDetailPage = (item) => {
  if (!item || !item.historyId) return
  router.push({ name: 'stock-history-detail', params: { historyId: item.historyId } })
}

onMounted(fetchHistories)
watch(page, fetchHistories)
</script>

<style scoped>
.title {
  font-size: 20px;
  font-weight: 600;
  color: #3f51b5;
}

.list {
  font-size: 16px;
  font-weight: 600;
  color: gray;
}

:deep(.rounded-table thead) {
  background-color: #f2f5f8;
}

:deep(.rounded-table tbody tr:hover) {
  background-color: #f4faff;
  cursor: pointer;
}

.hei {
  min-height: 900px;
}
</style>