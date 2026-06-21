<template>
  <div class="island-container">
    <div class="island-header">
      <h1>🏝️ 海岛生存</h1>
      <p>在荒岛上建立你的生存基地</p>
    </div>
    
    <div class="island-main">
      <div class="stats-panel">
        <div class="stat-card">
          <div class="stat-icon">🍖</div>
          <div class="stat-content">
            <div class="stat-number">{{ resources.food }}</div>
            <div class="stat-label">食物</div>
          </div>
        </div>
        
        <div class="stat-card">
          <div class="stat-icon">💧</div>
          <div class="stat-content">
            <div class="stat-number">{{ resources.water }}</div>
            <div class="stat-label">淡水</div>
          </div>
        </div>
        
        <div class="stat-card">
          <div class="stat-icon">🪵</div>
          <div class="stat-content">
            <div class="stat-number">{{ resources.wood }}</div>
            <div class="stat-label">木材</div>
          </div>
        </div>
        
        <div class="stat-card">
          <div class="stat-icon">⛏️</div>
          <div class="stat-content">
            <div class="stat-number">{{ resources.stone }}</div>
            <div class="stat-label">石头</div>
          </div>
        </div>
      </div>
      
      <div class="companion-panel">
        <h3>🐾 驯养伙伴</h3>
        <div class="companion-buffs" v-if="tamedCompanions.length > 0">
          <div class="buff-tag" v-if="scoutingLevel > 0">
            🔍 侦查 Lv.{{ scoutingLevel }}
          </div>
          <div class="buff-tag" v-if="carryingLevel > 0">
            🎒 搬运 Lv.{{ carryingLevel }}
          </div>
          <div class="buff-tag" v-if="alertLevel > 0">
            🛡️ 警戒 Lv.{{ alertLevel }}
          </div>
        </div>
        <div class="companion-section" v-if="tamedCompanions.length > 0">
          <h4>已驯养伙伴</h4>
          <div class="companion-grid">
            <div v-for="companion in tamedCompanions" :key="companion.id" class="companion-card tamed">
              <div class="companion-icon">{{ companion.icon }}</div>
              <div class="companion-info">
                <div class="companion-name">{{ companion.name }}</div>
                <div class="companion-ability">{{ companion.abilityLabel }}</div>
                <div class="companion-upkeep">每日消耗: {{ companion.upkeep }}食物</div>
              </div>
              <el-button size="small" type="danger" @click="releaseCompanion(companion)">放归</el-button>
            </div>
          </div>
        </div>
        <div class="companion-section">
          <h4>可驯养动物</h4>
          <div class="companion-grid">
            <div v-for="animal in availableWildAnimals" :key="animal.id" class="companion-card wild">
              <div class="companion-icon">{{ animal.icon }}</div>
              <div class="companion-info">
                <div class="companion-name">{{ animal.name }}</div>
                <div class="companion-ability">{{ animal.abilityLabel }}</div>
                <div class="companion-ability-desc">{{ animal.abilityDesc }}</div>
                <div class="companion-upkeep">驯养需要: {{ animal.tameCost }}食物 | 每日消耗: {{ animal.upkeep }}食物</div>
              </div>
              <el-button size="small" type="success" @click="tameAnimal(animal)" :disabled="resources.food < animal.tameCost">
                驯养
              </el-button>
            </div>
          </div>
        </div>
      </div>
      
      <div class="actions-panel">
        <h3>📋 可执行操作</h3>
        
        <div class="action-grid">
          <div class="action-card" @click="gatherFood">
            <div class="action-icon">🍓</div>
            <div class="action-title">采集食物</div>
            <div class="action-desc">在岛上寻找可食用的果实和动物</div>
            <div class="action-time">耗时: 30秒</div>
          </div>
          
          <div class="action-card" @click="collectWater">
            <div class="action-icon">💧</div>
            <div class="action-title">收集淡水</div>
            <div class="action-desc">收集雨水或净化海水</div>
            <div class="action-time">耗时: 1分钟</div>
          </div>
          
          <div class="action-card" @click="chopWood">
            <div class="action-icon">🪓</div>
            <div class="action-title">砍伐木材</div>
            <div class="action-desc">砍伐树木获取木材资源</div>
            <div class="action-time">耗时: 2分钟</div>
          </div>
          
          <div class="action-card" @click="mineStone">
            <div class="action-icon">⛏️</div>
            <div class="action-title">挖掘石头</div>
            <div class="action-desc">在岛上挖掘石头资源</div>
            <div class="action-time">耗时: 3分钟</div>
          </div>
          
          <div class="action-card" @click="buildShelter">
            <div class="action-icon">🏠</div>
            <div class="action-title">建造庇护所</div>
            <div class="action-desc">建造一个安全的住所</div>
            <div class="action-cost">需要: 50木材, 30石头</div>
          </div>
          
          <div class="action-card" @click="craftTools">
            <div class="action-icon">🔨</div>
            <div class="action-title">制作工具</div>
            <div class="action-desc">制作更高效的生存工具</div>
            <div class="action-cost">需要: 20木材, 10石头</div>
          </div>
        </div>
      </div>
      
      <div class="map-panel">
        <h3>🗺️ 海岛地图</h3>
        <div class="map-container">
          <div class="map-grid">
            <div v-for="(cell, index) in mapGrid" :key="index" 
                 :class="'map-cell ' + cell.type + (cell.explored ? ' explored' : '')"
                 @click="exploreCell(index)">
              {{ cell.icon }}
            </div>
          </div>
          <div class="map-legend">
            <div class="legend-item">
              <span class="legend-icon">🌳</span>
              <span>森林</span>
            </div>
            <div class="legend-item">
              <span class="legend-icon">🏔️</span>
              <span>山地</span>
            </div>
            <div class="legend-item">
              <span class="legend-icon">🌊</span>
              <span>海洋</span>
            </div>
            <div class="legend-item">
              <span class="legend-icon">🏠</span>
              <span>营地</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="message-log">
      <h3>📜 生存日志</h3>
      <div class="log-list">
        <div v-for="(msg, index) in messageLog" :key="index" class="log-item">
          <span class="log-time">{{ msg.time }}</span>
          <span class="log-content">{{ msg.content }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { ElMessage, ElMessageBox } from 'element-plus';

const ANIMAL_CATALOG = [
  {
    id: 'eagle',
    name: '鹰',
    icon: '🦅',
    ability: 'scouting',
    abilityLabel: '侦查',
    abilityDesc: '提升探索收益，降低遭遇危险的概率',
    tameCost: 40,
    upkeep: 3
  },
  {
    id: 'falcon',
    name: '猎隼',
    icon: '🦅',
    ability: 'scouting',
    abilityLabel: '侦查',
    abilityDesc: '大幅提升探索收益，降低遭遇危险的概率',
    tameCost: 60,
    upkeep: 5
  },
  {
    id: 'ox',
    name: '野牛',
    icon: '🐂',
    ability: 'carrying',
    abilityLabel: '搬运',
    abilityDesc: '降低资源采集消耗，提升采集产量',
    tameCost: 50,
    upkeep: 4
  },
  {
    id: 'mule',
    name: '骡子',
    icon: '🐴',
    ability: 'carrying',
    abilityLabel: '搬运',
    abilityDesc: '大幅降低资源采集消耗，提升采集产量',
    tameCost: 45,
    upkeep: 3
  },
  {
    id: 'hound',
    name: '猎犬',
    icon: '🐕',
    ability: 'alert',
    abilityLabel: '警戒',
    abilityDesc: '降低探索危险损失，减少营地被袭概率',
    tameCost: 35,
    upkeep: 3
  },
  {
    id: 'wolf',
    name: '狼',
    icon: '🐺',
    ability: 'alert',
    abilityLabel: '警戒',
    abilityDesc: '大幅降低探索危险损失，减少营地被袭概率',
    tameCost: 55,
    upkeep: 5
  }
];

const resources = ref({
  food: 100,
  water: 100,
  wood: 100,
  stone: 100
});

const messageLog = ref([
  { time: '00:00', content: '你来到了一个荒岛，开始你的生存之旅吧！' }
]);

const tamedCompanions = ref([]);

const scoutingLevel = computed(() => {
  return tamedCompanions.value.filter(c => c.ability === 'scouting').length;
});

const carryingLevel = computed(() => {
  return tamedCompanions.value.filter(c => c.ability === 'carrying').length;
});

const alertLevel = computed(() => {
  return tamedCompanions.value.filter(c => c.ability === 'alert').length;
});

const carryingMultiplier = computed(() => {
  return 1 + carryingLevel.value * 0.3;
});

const carryingDiscount = computed(() => {
  return 1 - Math.min(carryingLevel.value * 0.15, 0.45);
});

const dangerReduction = computed(() => {
  return Math.min(alertLevel.value * 0.25, 0.75);
});

const scoutingBonus = computed(() => {
  return 1 + scoutingLevel.value * 0.5;
});

const availableWildAnimals = computed(() => {
  const tamedIds = tamedCompanions.value.map(c => c.id);
  return ANIMAL_CATALOG.filter(a => !tamedIds.includes(a.id));
});

const mapGrid = ref([
  { type: 'forest', icon: '🌳', explored: true },
  { type: 'forest', icon: '🌳', explored: true },
  { type: 'mountain', icon: '🏔️', explored: false },
  { type: 'ocean', icon: '🌊', explored: false },
  { type: 'camp', icon: '🏠', explored: true },
  { type: 'forest', icon: '🌳', explored: false },
  { type: 'ocean', icon: '🌊', explored: false },
  { type: 'mountain', icon: '🏔️', explored: false },
  { type: 'forest', icon: '🌳', explored: false }
]);

const addMessage = (content) => {
  const now = new Date();
  const time = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
  messageLog.value.push({ time, content });
  if (messageLog.value.length > 20) {
    messageLog.value.shift();
  }
};

const tameAnimal = (animal) => {
  if (resources.value.food < animal.tameCost) {
    ElMessage.error(`食物不足，无法驯养${animal.name}！`);
    return;
  }
  ElMessageBox.confirm(
    `驯养${animal.name}需要${animal.tameCost}食物，每日消耗${animal.upkeep}食物。确定驯养吗？`,
    '驯养确认',
    {
      confirmButtonText: '确定驯养',
      cancelButtonText: '取消',
      type: 'info'
    }
  ).then(() => {
    resources.value.food -= animal.tameCost;
    tamedCompanions.value.push({ ...animal });
    addMessage(`成功驯养了${animal.icon}${animal.name}！获得${animal.abilityLabel}能力加成`);
    ElMessage.success(`驯养${animal.name}成功！`);
  }).catch(() => {});
};

const releaseCompanion = (companion) => {
  ElMessageBox.confirm(
    `确定要放归${companion.name}吗？你将失去它的${companion.abilityLabel}能力加成。`,
    '放归确认',
    {
      confirmButtonText: '确定放归',
      cancelButtonText: '取消',
      type: 'warning'
    }
  ).then(() => {
    const idx = tamedCompanions.value.findIndex(c => c.id === companion.id);
    if (idx !== -1) {
      tamedCompanions.value.splice(idx, 1);
    }
    addMessage(`放归了${companion.icon}${companion.name}，失去了${companion.abilityLabel}能力加成`);
    ElMessage.info(`${companion.name}已放归野外`);
  }).catch(() => {});
};

const performAction = (name, cost, gain, time) => {
  let adjustedCost = {};
  for (const [resource, amount] of Object.entries(cost)) {
    adjustedCost[resource] = Math.ceil(amount * carryingDiscount.value);
  }

  for (const [resource, amount] of Object.entries(adjustedCost)) {
    if (resources.value[resource] < amount) {
      ElMessage.error(`资源不足，无法${name}`);
      return false;
    }
  }

  for (const [resource, amount] of Object.entries(adjustedCost)) {
    resources.value[resource] -= amount;
  }

  addMessage(`开始${name}...`);

  setTimeout(() => {
    let adjustedGain = {};
    for (const [resource, amount] of Object.entries(gain)) {
      adjustedGain[resource] = Math.ceil(amount * carryingMultiplier.value);
    }
    for (const [resource, amount] of Object.entries(adjustedGain)) {
      resources.value[resource] += amount;
    }
    const gainDesc = Object.entries(adjustedGain).map(([k, v]) => `${v}${k}`).join('、');
    addMessage(`${name}完成！获得了${gainDesc}`);
    ElMessage.success(`${name}完成！`);
  }, time);

  return true;
};

const gatherFood = () => {
  performAction('采集食物', {}, { food: 20 }, 30000);
};

const collectWater = () => {
  performAction('收集淡水', {}, { water: 30 }, 60000);
};

const chopWood = () => {
  performAction('砍伐木材', {}, { wood: 15 }, 120000);
};

const mineStone = () => {
  performAction('挖掘石头', {}, { stone: 10 }, 180000);
};

const buildShelter = () => {
  if (performAction('建造庇护所', { wood: 50, stone: 30 }, {}, 300000)) {
    addMessage('庇护所建造完成！你现在有了一个安全的住所。');
  }
};

const craftTools = () => {
  if (performAction('制作工具', { wood: 20, stone: 10 }, {}, 120000)) {
    addMessage('工具制作完成！你的工作效率提高了。');
  }
};

const exploreCell = (index) => {
  const cell = mapGrid.value[index];
  if (cell.explored) {
    ElMessage.info('这个区域已经探索过了');
    return;
  }

  ElMessageBox.confirm(
    `确定要探索这个区域吗？可能会遇到危险或发现资源。`,
    '探索未知区域',
    {
      confirmButtonText: '开始探索',
      cancelButtonText: '取消',
      type: 'warning'
    }
  ).then(() => {
    addMessage(`开始探索${cell.icon}区域...`);

    setTimeout(() => {
      cell.explored = true;

      const random = Math.random();
      const dangerThreshold = 0.8 + scoutingLevel.value * 0.05;

      if (random < 0.3) {
        const baseGain = Math.floor(Math.random() * 20) + 10;
        const foodGain = Math.ceil(baseGain * scoutingBonus.value);
        resources.value.food += foodGain;
        addMessage(`探索发现了食物！获得${foodGain}食物`);
        ElMessage.success(`探索发现了食物！获得${foodGain}食物`);
      } else if (random < 0.6) {
        const baseGain = Math.floor(Math.random() * 15) + 5;
        const woodGain = Math.ceil(baseGain * scoutingBonus.value);
        resources.value.wood += woodGain;
        addMessage(`探索发现了木材！获得${woodGain}木材`);
        ElMessage.success(`探索发现了木材！获得${woodGain}木材`);
      } else if (random < dangerThreshold) {
        const baseGain = Math.floor(Math.random() * 10) + 5;
        const stoneGain = Math.ceil(baseGain * scoutingBonus.value);
        resources.value.stone += stoneGain;
        addMessage(`探索发现了石头！获得${stoneGain}石头`);
        ElMessage.success(`探索发现了石头！获得${stoneGain}石头`);
      } else {
        if (alertLevel.value > 0 && Math.random() < alertLevel.value * 0.3) {
          addMessage(`🛡️ 警戒伙伴发现了危险并发出警告！你及时避开了`);
          ElMessage.success('警戒伙伴发现了危险，你安全避开了！');
          const bonusFood = Math.floor(Math.random() * 10) + 5;
          resources.value.food += bonusFood;
          addMessage(`侦查伙伴带回了额外的${bonusFood}食物`);
        } else {
          const baseFoodLoss = 10;
          const baseWaterLoss = 10;
          const foodLoss = Math.ceil(baseFoodLoss * (1 - dangerReduction.value));
          const waterLoss = Math.ceil(baseWaterLoss * (1 - dangerReduction.value));
          resources.value.food -= foodLoss;
          resources.value.water -= waterLoss;
          const reducedMsg = dangerReduction.value > 0
            ? `（警戒伙伴为你减少了${Math.round(dangerReduction.value * 100)}%损失）`
            : '';
          addMessage(`探索遇到了危险！损失了${foodLoss}食物和${waterLoss}水${reducedMsg}`);
          ElMessage.warning(`探索遇到了危险！损失了${foodLoss}食物和${waterLoss}水`);
        }
      }
    }, 5000);
  }).catch(() => {
    addMessage('取消了探索');
  });
};

let consumptionTimer = null;

onMounted(() => {
  addMessage('欢迎来到海岛生存游戏！');
  consumptionTimer = setInterval(() => {
    const companionUpkeep = tamedCompanions.value.reduce((sum, c) => sum + c.upkeep, 0);
    const baseFoodDrain = 5;
    const baseWaterDrain = 5;
    const foodDrain = Math.ceil((baseFoodDrain + companionUpkeep) * carryingDiscount.value);
    const waterDrain = Math.ceil(baseWaterDrain * carryingDiscount.value);

    resources.value.food -= foodDrain;
    resources.value.water -= waterDrain;

    if (companionUpkeep > 0) {
      addMessage(`伙伴消耗了${companionUpkeep}食物${carryingLevel.value > 0 ? '（搬运加成减少消耗）' : ''}`);
    }

    if (alertLevel.value > 0 && Math.random() < 0.15) {
      addMessage(`🛡️ 警戒伙伴发现了潜在威胁并驱赶了它，营地安全了`);
    }

    if (resources.value.food <= 0 || resources.value.water <= 0) {
      ElMessageBox.alert(
        '你的食物或水耗尽了，游戏结束！',
        '游戏结束',
        {
          confirmButtonText: '重新开始',
          type: 'error'
        }
      ).then(() => {
        resources.value.food = 100;
        resources.value.water = 100;
        resources.value.wood = 100;
        resources.value.stone = 100;
        tamedCompanions.value = [];
        addMessage('重新开始游戏！');
      });
    }
  }, 60000);
});

onUnmounted(() => {
  if (consumptionTimer) {
    clearInterval(consumptionTimer);
    consumptionTimer = null;
  }
});
</script>

<style scoped>
.island-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.island-header {
  text-align: center;
  color: white;
  margin-bottom: 30px;
}

.island-header h1 {
  font-size: 48px;
  margin: 0 0 10px 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.island-header p {
  font-size: 18px;
  margin: 0;
  opacity: 0.9;
}

.island-main {
  max-width: 1200px;
  margin: 0 auto;
}

.stats-panel {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.stat-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  align-items: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.stat-icon {
  font-size: 48px;
  margin-right: 20px;
}

.stat-content {
  flex: 1;
}

.stat-number {
  font-size: 32px;
  font-weight: bold;
  color: #333;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 14px;
  color: #666;
}

.companion-panel {
  background: white;
  border-radius: 12px;
  padding: 30px;
  margin-bottom: 30px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.companion-panel h3 {
  margin: 0 0 20px 0;
  font-size: 24px;
  color: #333;
}

.companion-buffs {
  display: flex;
  gap: 12px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.buff-tag {
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 600;
  color: white;
}

.buff-tag:nth-child(1) {
  background: linear-gradient(135deg, #43e97b, #38f9d7);
}

.buff-tag:nth-child(2) {
  background: linear-gradient(135deg, #fa709a, #fee140);
}

.buff-tag:nth-child(3) {
  background: linear-gradient(135deg, #667eea, #764ba2);
}

.companion-section {
  margin-bottom: 20px;
}

.companion-section:last-child {
  margin-bottom: 0;
}

.companion-section h4 {
  margin: 0 0 12px 0;
  font-size: 16px;
  color: #555;
  border-bottom: 1px solid #eee;
  padding-bottom: 8px;
}

.companion-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 16px;
}

.companion-card {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 16px;
  border-radius: 10px;
  transition: all 0.3s ease;
  border: 2px solid transparent;
}

.companion-card.tamed {
  background: linear-gradient(135deg, #e8f8f0, #d4efdf);
  border-color: #43e97b;
}

.companion-card.wild {
  background: #f8f9fa;
}

.companion-card.wild:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.08);
  border-color: #667eea;
}

.companion-icon {
  font-size: 42px;
  min-width: 50px;
  text-align: center;
}

.companion-info {
  flex: 1;
}

.companion-name {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 4px;
}

.companion-ability {
  font-size: 13px;
  font-weight: 600;
  color: #667eea;
  margin-bottom: 2px;
}

.companion-ability-desc {
  font-size: 12px;
  color: #888;
  margin-bottom: 4px;
}

.companion-upkeep {
  font-size: 12px;
  color: #e6a23c;
}

.actions-panel {
  background: white;
  border-radius: 12px;
  padding: 30px;
  margin-bottom: 30px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.actions-panel h3 {
  margin: 0 0 20px 0;
  font-size: 24px;
  color: #333;
}

.action-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 20px;
}

.action-card {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
}

.action-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  border-color: #667eea;
}

.action-icon {
  font-size: 48px;
  margin-bottom: 12px;
}

.action-title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 8px;
}

.action-desc {
  font-size: 14px;
  color: #666;
  margin-bottom: 8px;
}

.action-time,
.action-cost {
  font-size: 12px;
  color: #999;
}

.map-panel {
  background: white;
  border-radius: 12px;
  padding: 30px;
  margin-bottom: 30px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.map-panel h3 {
  margin: 0 0 20px 0;
  font-size: 24px;
  color: #333;
}

.map-container {
  text-align: center;
}

.map-grid {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  gap: 10px;
  justify-content: center;
  margin-bottom: 30px;
}

.map-cell {
  width: 100px;
  height: 100px;
  background: #f0f0f0;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid #ddd;
}

.map-cell:hover {
  transform: scale(1.05);
  border-color: #667eea;
}

.map-cell.explored {
  background: #e8f4fa;
  border-color: #409eff;
}

.map-legend {
  display: flex;
  justify-content: center;
  gap: 30px;
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
}

.legend-icon {
  font-size: 24px;
}

.message-log {
  background: white;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.message-log h3 {
  margin: 0 0 20px 0;
  font-size: 24px;
  color: #333;
}

.log-list {
  max-height: 300px;
  overflow-y: auto;
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 10px;
}

.log-item {
  display: flex;
  margin-bottom: 8px;
  padding: 8px;
  background: #f8f9fa;
  border-radius: 4px;
}

.log-time {
  font-weight: bold;
  color: #409eff;
  margin-right: 12px;
  min-width: 60px;
}

.log-content {
  flex: 1;
  color: #666;
}

@media (max-width: 768px) {
  .island-header h1 {
    font-size: 32px;
  }
  
  .stats-panel {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .action-grid {
    grid-template-columns: 1fr;
  }
}
</style>