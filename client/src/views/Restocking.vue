<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <div class="card budget-card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.budgetLabel') }}</h3>
          <div class="budget-value">{{ currencySymbol }}{{ budget.toLocaleString() }}</div>
        </div>
        <div class="slider-row">
          <input
            type="range"
            min="0"
            max="500000"
            step="1000"
            v-model.number="budget"
            class="budget-slider"
          />
          <div class="slider-ticks">
            <span>{{ currencySymbol }}0</span>
            <span>{{ currencySymbol }}500K</span>
          </div>
        </div>
        <div class="budget-summary">
          <div class="summary-item">
            <span class="summary-label">{{ t('restocking.totalCost') }}</span>
            <span class="summary-value cost">{{ currencySymbol }}{{ totalCost.toLocaleString() }}</span>
          </div>
          <div class="summary-item">
            <span class="summary-label">{{ t('restocking.remaining') }}</span>
            <span class="summary-value remaining" :class="{ negative: remaining < 0 }">
              {{ currencySymbol }}{{ remaining.toLocaleString() }}
            </span>
          </div>
          <div class="summary-item">
            <span class="summary-label">{{ t('restocking.recommended') }}</span>
            <span class="summary-value">{{ recommended.length }}</span>
          </div>
        </div>
        <div class="action-row">
          <button
            class="place-order-btn"
            :disabled="recommended.length === 0 || placing"
            @click="placeOrder"
          >
            {{ placing ? t('restocking.placing') : t('restocking.placeOrder') }}
          </button>
          <div v-if="placedMessage" class="success-banner">{{ placedMessage }}</div>
        </div>
      </div>

      <div class="card">
        <div class="card-header">
          <h3 class="card-title">
            {{ t('restocking.recommended') }} ({{ recommended.length }})
          </h3>
        </div>
        <div v-if="recommended.length === 0" class="empty-state">
          {{ t('restocking.noRecommendations') }}
        </div>
        <div v-else class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('restocking.columns.sku') }}</th>
                <th>{{ t('restocking.columns.name') }}</th>
                <th>{{ t('restocking.columns.onHand') }}</th>
                <th>{{ t('restocking.columns.reorderPoint') }}</th>
                <th>{{ t('restocking.columns.forecasted') }}</th>
                <th>{{ t('restocking.columns.qtyToOrder') }}</th>
                <th>{{ t('restocking.columns.unitCost') }}</th>
                <th>{{ t('restocking.columns.lineTotal') }}</th>
                <th>{{ t('restocking.columns.reason') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommended" :key="item.sku">
                <td><strong>{{ item.sku }}</strong></td>
                <td>{{ translateProductName(item.name) }}</td>
                <td>{{ item.quantity_on_hand }}</td>
                <td>{{ item.reorder_point }}</td>
                <td>{{ item.forecasted_demand }}</td>
                <td><strong>{{ item.suggested_qty }}</strong></td>
                <td>{{ currencySymbol }}{{ item.unit_cost.toFixed(2) }}</td>
                <td>{{ currencySymbol }}{{ item.line_total.toLocaleString() }}</td>
                <td>
                  <span :class="['badge', item.reason === 'lowStock' ? 'danger' : 'warning']">
                    {{ item.reason === 'lowStock' ? t('restocking.reasonLowStock') : t('restocking.reasonRising') }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div v-if="skipped.length > 0" class="card skipped-card">
        <div class="card-header">
          <h3 class="card-title">
            {{ t('restocking.skipped') }} ({{ skipped.length }})
          </h3>
        </div>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('restocking.columns.sku') }}</th>
                <th>{{ t('restocking.columns.name') }}</th>
                <th>{{ t('restocking.columns.qtyToOrder') }}</th>
                <th>{{ t('restocking.columns.unitCost') }}</th>
                <th>{{ t('restocking.columns.lineTotal') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in skipped" :key="item.sku">
                <td><strong>{{ item.sku }}</strong></td>
                <td>{{ translateProductName(item.name) }}</td>
                <td>{{ item.suggested_qty }}</td>
                <td>{{ currencySymbol }}{{ item.unit_cost.toFixed(2) }}</td>
                <td>{{ currencySymbol }}{{ item.line_total.toLocaleString() }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch, computed } from 'vue'
import { useRouter } from 'vue-router'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const router = useRouter()
    const { t, currentCurrency, translateProductName } = useI18n()
    const currencySymbol = computed(() => currentCurrency.value === 'JPY' ? '¥' : '$')

    const loading = ref(true)
    const error = ref(null)
    const inventoryItems = ref([])
    const forecasts = ref([])
    const budget = ref(50000)
    const placing = ref(false)
    const placedMessage = ref('')

    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    const loadData = async () => {
      try {
        loading.value = true
        error.value = null
        const filters = getCurrentFilters()
        const [forecastData, inventoryData] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory({
            warehouse: filters.warehouse,
            category: filters.category,
          })
        ])
        forecasts.value = forecastData
        inventoryItems.value = inventoryData
      } catch (err) {
        error.value = 'Failed to load restocking data: ' + err.message
      } finally {
        loading.value = false
      }
    }

    // Join forecasts with inventory on item_sku === sku, compute suggested qty
    const candidates = computed(() => {
      const inventoryBySku = new Map(inventoryItems.value.map(i => [i.sku, i]))
      const result = []
      for (const f of forecasts.value) {
        const inv = inventoryBySku.get(f.item_sku)
        if (!inv) continue
        const suggested = Math.max(0, f.forecasted_demand - inv.quantity_on_hand)
        if (suggested === 0) continue
        result.push({
          sku: inv.sku,
          name: inv.name,
          quantity_on_hand: inv.quantity_on_hand,
          reorder_point: inv.reorder_point,
          unit_cost: inv.unit_cost,
          current_demand: f.current_demand,
          forecasted_demand: f.forecasted_demand,
          suggested_qty: suggested,
          line_total: Math.round(suggested * inv.unit_cost * 100) / 100,
          reason: inv.quantity_on_hand <= inv.reorder_point ? 'lowStock' : 'rising',
        })
      }
      return result
    })

    // Priority: below reorder_point first, then largest demand growth
    const prioritized = computed(() => {
      return [...candidates.value].sort((a, b) => {
        const aReason = a.reason === 'lowStock' ? 0 : 1
        const bReason = b.reason === 'lowStock' ? 0 : 1
        if (aReason !== bReason) return aReason - bReason
        const aGrowth = a.forecasted_demand - a.current_demand
        const bGrowth = b.forecasted_demand - b.current_demand
        return bGrowth - aGrowth
      })
    })

    // Greedy fill: include items whose running total stays within budget
    const partition = computed(() => {
      const recommended = []
      const skipped = []
      let running = 0
      for (const item of prioritized.value) {
        if (running + item.line_total <= budget.value) {
          recommended.push(item)
          running += item.line_total
        } else {
          skipped.push(item)
        }
      }
      return { recommended, skipped, running }
    })

    const recommended = computed(() => partition.value.recommended)
    const skipped = computed(() => partition.value.skipped)
    const totalCost = computed(() => Math.round(partition.value.running * 100) / 100)
    const remaining = computed(() => Math.round((budget.value - totalCost.value) * 100) / 100)

    const placeOrder = async () => {
      if (recommended.value.length === 0 || placing.value) return
      try {
        placing.value = true
        placedMessage.value = ''
        const filters = getCurrentFilters()
        const payload = {
          items: recommended.value.map(c => ({
            sku: c.sku,
            name: c.name,
            quantity: c.suggested_qty,
            unit_price: c.unit_cost,
          })),
          warehouse: filters.warehouse !== 'all' ? filters.warehouse : null,
          category: filters.category !== 'all' ? filters.category : null,
        }
        await api.createOrder(payload)
        placedMessage.value = t('restocking.orderSuccess')
        setTimeout(() => router.push('/orders'), 1500)
      } catch (err) {
        error.value = 'Failed to place order: ' + err.message
      } finally {
        placing.value = false
      }
    }

    watch([selectedLocation, selectedCategory], loadData)
    onMounted(loadData)

    return {
      t,
      loading,
      error,
      budget,
      placing,
      placedMessage,
      recommended,
      skipped,
      totalCost,
      remaining,
      currencySymbol,
      translateProductName,
      placeOrder,
    }
  },
}
</script>

<style scoped>
.restocking {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.budget-card .card-header {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
}

.budget-value {
  font-size: 1.75rem;
  font-weight: 700;
  color: #0f172a;
  font-variant-numeric: tabular-nums;
}

.slider-row {
  padding: 0.5rem 1rem 1rem;
}

.budget-slider {
  width: 100%;
  -webkit-appearance: none;
  appearance: none;
  height: 6px;
  background: #e2e8f0;
  border-radius: 3px;
  outline: none;
}

.budget-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: #3b82f6;
  border: 3px solid white;
  box-shadow: 0 1px 3px rgba(15, 23, 42, 0.2);
  cursor: pointer;
}

.budget-slider::-moz-range-thumb {
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: #3b82f6;
  border: 3px solid white;
  box-shadow: 0 1px 3px rgba(15, 23, 42, 0.2);
  cursor: pointer;
}

.slider-ticks {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #64748b;
  margin-top: 0.5rem;
}

.budget-summary {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  padding: 1rem;
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  border-bottom: 1px solid #e2e8f0;
}

.summary-item {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.summary-label {
  font-size: 0.75rem;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  font-weight: 600;
}

.summary-value {
  font-size: 1.25rem;
  font-weight: 600;
  color: #0f172a;
  font-variant-numeric: tabular-nums;
}

.summary-value.cost { color: #0f172a; }
.summary-value.remaining { color: #10b981; }
.summary-value.remaining.negative { color: #ef4444; }

.action-row {
  display: flex;
  gap: 1rem;
  align-items: center;
  padding: 1rem;
}

.place-order-btn {
  background: #3b82f6;
  color: white;
  font-weight: 600;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 6px;
  font-size: 0.9375rem;
  cursor: pointer;
  transition: background 0.15s ease;
}

.place-order-btn:hover:not(:disabled) {
  background: #2563eb;
}

.place-order-btn:disabled {
  background: #cbd5e1;
  cursor: not-allowed;
}

.success-banner {
  flex: 1;
  padding: 0.5rem 0.75rem;
  background: #d1fae5;
  color: #065f46;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 500;
}

.empty-state {
  padding: 2.5rem;
  text-align: center;
  color: #64748b;
  font-size: 0.9375rem;
}

.skipped-card {
  opacity: 0.92;
}

.skipped-card .card-title {
  color: #64748b;
}
</style>
