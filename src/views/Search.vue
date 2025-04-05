<template>
<header class="header">
        <div class="header-content">
            <div class="logo-container">
                <div class="logo-todedit">TODEDIT</div>
                <div class="logo-ijournal">ijournal</div>
            </div>
            
            <nav class="nav-menu">
                <a href="#" class="nav-item">期刊详情查询</a>
                <a href="#" class="nav-item">AI智能选刊</a>
                <a href="#" class="nav-item">期刊解析与推荐</a>
                <a href="#" class="nav-item">iThenticate查重</a>
                <a href="#" class="nav-item">关于我们</a>
            </nav>
            
            <button class="login-btn">登录/注册</button>
        </div>
</header>

<!-- 搜索和过滤区域(连在一起) -->
<section class="search-section">
  <!-- 搜索区域 -->
  <div class="search-container">
      <div class="search-box">
          <select class="search-type">
              <option>期刊名称</option>
          </select>
          <input type="text" class="search-input" placeholder="请输入内容">
          <button class="search-button">
              <i>🔍</i>
          </button>
      </div>
  </div>
  
  <!-- 过滤选项 -->
  <div class="filter-container">
  <div class="filter-item">
    <span class="filter-label">影响因子</span>
    <input 
      v-model="ifStart_2019" 
      type="text" 
      class="filter-input"
      @blur="handleStartBlur">
    <span class="separator-corridor">-</span>
    <input 
      v-model="ifEnd_2019" 
      type="text" 
      class="filter-input"
      @blur="handleEndBlur"
    >
    </div>
      
    <div class="filter-item">
    <span class="filter-label">JCR分区</span>
      <div class="dropdown" @click.stop="jcrToggleDropdown">
      {{ jcrSelectedOption || '不限' }}
        <div v-if="jcrIsDropdownOpen" class="dropdown-menu">
          <div class="dropdown-item" v-for="(option, index) in jcrOptions" :key="index" @click.stop="jcrSelectOption(option)">
            {{ option }}
          </div>
        </div>
      </div>
    </div>

    <div class="filter-item">
    <span class="filter-label">分区</span>
      <div class="dropdown" @click.stop="toggleDropdown">
      {{ selectedOption || '不限' }}
      <div v-if="isDropdownOpen" class="dropdown-menu">
        <div class="dropdown-item" v-for="(option, index) in options" :key="index" @click.stop="selectOption(option)">
          {{ option }}
        </div>
      </div>
    </div>
    </div>
      
    <div class="filter-item">
      <span class="filter-label">综述占比</span>
          <div class="dropdown" @click.stop="synthesisDropdown">
          {{ synthesisSelectedOption || '不限' }}
          <div v-if="synthesisDropdownOpen" class="dropdown-menu">
          <div class="dropdown-item" v-for="(option, index) in synthesisPercentageOptions" :key="index" @click.stop="synthesisOptions(option)">
            {{ option }}
          </div>
        </div>
      </div>
    </div>
  </div>

</section>

<!-- 表格部分 -->
<section class="journal-table-container">
  <table class="journal-table">
      <thead>
          <tr>
              <th class="hover-text">
                期刊名称
                <span class="sort-icon"></span>
              </th>
              <th class="hover-text" @click="handleSort('journalDb')">
                数据类
                <span class="sort-icon" v-html="getSortIcon('journalDb')"></span>
              </th>
              <th class="hover-text" @click="handleSort('impactFactor_2019')">
                影响因子
                <span class="sort-icon" v-html="getSortIcon('impactFactor_2019')"></span>
              </th>
              <th class="hover-text" @click="handleSort('quote_factor')">
                引文指标
                <span class="sort-icon" v-html="getSortIcon('quote_factor')"></span>
              </th>
              <th class="hover-text">
                JCR分区
                <span class="sort-icon"></span>
              </th>
              <th class="hover-text" @click="handleSort('subZone')">
                国内分区
                <span class="sort-icon" v-html="getSortIcon('subZone')"></span>
              </th>
              <th class="hover-text" @click="handleSort('annualPublication')">
                年发文章
                <span class="sort-icon" v-html="getSortIcon('annualPublication')"></span>
              </th>
              <th class="hover-text" @click="handleSort('totalReviewRatio')">
                综述占比
                <span class="sort-icon" v-html="getSortIcon('totalReviewRatio')"></span>
              </th>
              <th class="hover-text" @click="handleSort('open_ratio')">
                是否开源
                <span class="sort-icon" v-html="getSortIcon('open_ratio')"></span>
              </th>
              <th class="hover-text">
                是否国产
                <span class="sort-icon"></span>
              </th>
          </tr>
      </thead>
      <tbody>
          <tr v-for="(journal, index) in journalData" :key="index">
              <td class="journal-name">{{ journal.journalTitle }}<span class="journal-abbr">{{ journal.abbrJournal }}</span></td>
              <td>{{ journal.journalDb }}</td>
              <td>{{ journal.impactFactor_2019 }}</td>
              <td>{{ journal.quote_factor }}</td>
              <td>{{ getJcrSub(journal.jcr_Json) }}</td>
              <td>{{ journal.subZone > 0 ? journal.subZone + '区' : '-' }}</td>
              <td>{{ journal.annualPublication > 0 ? journal.annualPublication : '-' }}</td>
              <td>{{ journal.totalReviewRatio === null ? '-' : journal.totalReviewRatio + '%' }}</td>
              <td>{{ journal.open_ratio>0 ? ((journal.open_ratio*100).toFixed(2) + '%') : '0.00%' }}</td>
              <td>{{journal.isDomestic === 0 ? '否' : '是'}}</td>
          </tr>
      </tbody>
  </table>
  
  <el-pagination
    v-model:current-page="currentPage"
    :page-size="pageSize"
    :total="total"
    :pager-count="7"
    layout="prev, pager, next, jumper"
    @current-change="handleCurrentChange"
    background
    hide-on-single-page
    class="pagination"
  />
</section>


<!-- 页脚部分 -->
<footer class="footer">
  <div class="footer-content">
    <!-- 文字 -->
    <div class="logo-container">
      <span class="todedit">TODEDIT</span>
      <span class="separator">|</span>
      <span class="ijournal">iJOUrnAL</span>
    </div>
    <div class="footer-columns">
      <div class="footer-column">
        <h3>iJournal平台</h3>
        <a href="#" class="footer-link">首页</a>
        <a href="#" class="footer-link">关于我们</a>
      </div>
      
      <div class="footer-column">
        <h3>相关服务</h3>
        <a href="#" class="footer-link">论文服务</a>
        <a href="#" class="footer-link">查重服务</a>
      </div>
      
      <div class="footer-column">
        <h3>战略合作伙伴</h3>
        <a href="#" class="footer-link">投必得学术公众号</a>
        <a href="#" class="footer-link">投必得学术知乎机构号</a>
        <a href="#" class="footer-link">智慧科研公众号</a>
      </div>
      
      <div class="footer-column">
        <h3>联系我们</h3>
        <a href="#" class="footer-link">iJournal平台客服</a>
      </div>
    </div>
  </div>
  
  <div class="footer-bottom">
    <p class="copyright">© 2019 Todedit All Rights Reserved 
    <img src="@/assets/ssl.svg" alt="网络安全认证">
    </p>
  </div>
</footer>
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted, onUnmounted, watch } from 'vue';
import { ElMessage,ElLoading } from 'element-plus';
import 'element-plus/dist/index.css';

// 定义下拉选项
const jcrOptions = ref(['不限', 'Q1', 'Q2', 'Q3', 'Q4']);
const options = ref(['不限', '1区', '2区', '3区', '4区']);
const synthesisPercentageOptions = ref(['不限', '0-10%', '10-20%', '20-30%', '30-40%', '40-50%', '50%以上']);

// 定义下拉菜单的状态
const jcrIsDropdownOpen = ref(false); 
const isDropdownOpen = ref(false);
const synthesisDropdownOpen = ref(false); 


// 为每个下拉菜单创建独立的选中选项变量
const jcrSelectedOption = ref('不限'); 
const selectedOption = ref('不限'); 
const synthesisSelectedOption = ref('不限');


// 切换下拉菜单显示状态
const jcrToggleDropdown = () => {
  jcrIsDropdownOpen.value = !jcrIsDropdownOpen.value;
};
const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
};
const synthesisDropdown = () => {
  synthesisDropdownOpen.value = !synthesisDropdownOpen.value;
};

// 选择选项,并关闭下拉菜单
const jcrSelectOption = (option) => {
  jcrSelectedOption.value = option;
  jcrIsDropdownOpen.value = false;
};

const selectOption = (option) => {
  selectedOption.value = option;
  isDropdownOpen.value = false;
};

const synthesisOptions = (option) => {
  synthesisSelectedOption.value = option;
  synthesisDropdownOpen.value = false;
};

// 点击外部关闭下拉菜单
const closeDropdownOnOutsideClick = (e) => {
  if (jcrIsDropdownOpen.value && !e.target.closest('.dropdown')) {
    jcrIsDropdownOpen.value = false;
  }
  if (isDropdownOpen.value && !e.target.closest('.dropdown')) {
    isDropdownOpen.value = false;
  }
  if (synthesisDropdownOpen.value && !e.target.closest('.dropdown')) {
    synthesisDropdownOpen.value = false;
  }
};

// 定义本地变量存储请求参数
const pageNum = ref(1);
const pageSize = ref(10);
const keywordType = ref('title');
const keyword = ref('');
const ifStart_2019 = ref(''); // 影响因子开始参数
const ifEnd_2019 = ref('');   // 影响因子结束参数
const ifStart = ref('');
const ifEnd = ref('');
const jcr = ref('');          // jcr参数
const sub = ref('');          // 分区参数
const selfCitingRate = ref('all');
const compatriotRate = ref('all');
const isDomestic = ref('');
const totalReviewRatio = ref(''); // 综述参数
const categoryId = ref('1e78b26454e2878930f44c6a571be497');
const recommend = ref(0);
const selfStart = ref('');
const selfEnd = ref('');
const numberStart = ref('');
const numberEnd = ref('');
const trrl = ref('');
const trrr = ref('');
const order = ref('');
const orderType = ref('');

// 存储返回的数据
const journalData = ref([]);
const loading = ref(false);
const total = ref(0);
const currentPage = ref(1);
const lastPage = ref(Math.ceil(total.value / 10));

// 定义影响因子的初始值
const ifStartValid = ref(false);
const ifEndValid = ref(false);

// 处理起始值输入框失去焦点
const handleStartBlur = () => {
  // 验证是否为有效数字
  ifStartValid.value = validateNumber(ifStart_2019.value);
  if(!ifStartValid.value){
    ElMessage({
      message: '请输入有效的起始影响因子值',
      type: 'warning',
      center: true, // 这会使消息居中显示
      duration: 2000 // 显示2秒
  });
}};

// 处理结束值输入框失去焦点
const handleEndBlur = () => {
  // 验证是否为有效数字
  ifEndValid.value = validateNumber(ifEnd_2019.value);
  if(ifEndValid.value){
    if(ifStartValid.value && ifEndValid.value){
      fetchJournalData();
    }}
  else{
    ElMessage({
      message: '请输入有效的结束影响因子值',
      type: 'warning',
      center: true, // 这会使消息居中显示
      duration: 2000 // 显示2秒
  })}
};

// 分页逻辑
const handleCurrentChange = (page) => {
  // 这里添加获取新页面数据的逻辑
  pageNum.value = page;
  fetchJournalData(page);
};


// loading 处理函数









// JCR分区与数值的映射关系
const jcrValueMap = {
  'Q1': 8,
  'Q2': 4,
  'Q3': 2,
  'Q4': 1,
  '不限': ''
};

// 分区映射
const subValueMap = {
  '1区': 1,
  '2区': 2,
  '3区': 3,
  '4区': 4,
  '不限': ''
};

// 综述映射
const synthesisValueMap = {
  '不限': 'all',
  '0-10%': 1,
  '10-20%': 2,
  '20-30%': 3,
  '30-40%': 4,
  '40-50%': 5,
  '50%以上': 6
};

// 排序处理函数 - 新增
const handleSort = (column) => {
  if (order.value === column) {
    // 如果点击当前排序的列，则循环切换 "默认 -> 升序 -> 降序 -> 默认"
    if (orderType.value === '') {
      orderType.value = 'asc';
    } else if (orderType.value === 'asc') {
      orderType.value = 'desc';
    } else {
      orderType.value = '';
    }
  } else {
    // 如果点击新的列，设置为升序状态
    order.value = column;
    orderType.value = 'asc';
  }
  
  // console.log(`排序列: ${order.value}, 排序方式: ${orderType.value}`);
  
  // 如果需要后端排序，取消注释下面的代码重新获取数据
  // fetchJournalData();
};

// 获取排序图标 - 新增
// 修改获取排序图标函数，使用SVG实现带间距的上下三角
const getSortIcon = (column) => {
  // 默认显示灰色上下三角，尺寸从16x16调整为20x20
  if (order.value !== column || orderType.value === '') {
    return '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 16 16">' + 
           '<polygon points="8,4 12,7 4,7" fill="#CCCCCC"/>' + 
           '<polygon points="8,12 4,9 12,9" fill="#CCCCCC"/></svg>';
  }
  if (orderType.value === 'asc') {
    return '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 16 16">' + 
           '<polygon points="8,4 12,7 4,7" fill="#00B0B0"/>' + 
           '<polygon points="8,12 4,9 12,9" fill="#CCCCCC"/></svg>';
  } else if (orderType.value === 'desc') {
    return '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 16 16">' + 
           '<polygon points="8,4 12,7 4,7" fill="#CCCCCC"/>' + 
           '<polygon points="8,12 4,9 12,9" fill="#00B0B0"/></svg>';
  }
};;

// 请求数据的函数
const fetchJournalData = async () => {
  // 获取表格容器元素
  const tableContainer = document.querySelector('.journal-table-container');
  const loadingInstance = ElLoading.service({
    target: tableContainer, // 指定loading效果显示的DOM节点
    lock: true,
    text: '加载中...',
    background: 'rgba(255, 255, 255, 0.9)'
  })
  try {
    const response = await axios.get('https://test-ijournal-service.topeditsci.com/api/v1/journal/search', {
      params: {
        pageNum: pageNum.value,
        pageSize: pageSize.value,
        total: total.value,
        keywordType: keywordType.value,
        keyword: keyword.value,
        ifStart_2019: ifStart_2019.value,
        ifEnd_2019: ifEnd_2019.value,
        ifStart: ifStart.value,
        ifEnd: ifEnd.value,
        jcr: jcr.value,
        sub: sub.value,
        selfCitingRate: selfCitingRate.value,
        compatriotRate: compatriotRate.value,
        isDomestic: isDomestic.value,
        totalReviewRatio: totalReviewRatio.value,
        categoryId: categoryId.value,
        recommend: recommend.value,
        selfStart: selfStart.value,
        selfEnd: selfEnd.value,
        numberStart: numberStart.value,
        numberEnd: numberEnd.value,
        trrl: trrl.value,
        trrr: trrr.value,
        order: order.value,
        orderType: orderType.value
      }
    });
    
    // 根据返回的数据结构，正确获取期刊数据
    if (response.data && response.data.code === 0 && response.data.data_original) {
      // 获取期刊列表数据
      journalData.value = response.data.data_original.data || [];
      // console.log(journalData);
      
      // 获取分页信息
      total.value = response.data.data_original.total || 0;
      currentPage.value = response.data.data_original.currentPage || 1;
      lastPage.value = response.data.data_original.lastPage || 1;
    } else {
      console.error('返回数据格式不正确:', response.data);
      journalData.value = [];
    }
  } catch (error) {
    console.error('获取期刊数据失败:', error);
    journalData.value = [];
  } finally {
    loadingInstance.close();
  }
};

// json数据处理
const getJcrSub = (jcrJson) => {
  try {
    // 如果jcr_Json是字符串，先解析它
    const jcrData = typeof jcrJson === 'string' ? JSON.parse(jcrJson) : jcrJson;
    
    // 检查是否有数据
    if (jcrData && jcrData.length > 0 && jcrData[0].sub) {
      return jcrData[0].sub;
    }
    
    // 如果没有数据或数据格式不对，返回默认值
    return '-';
  } catch (error) {
    console.error('Error parsing JCR data:', error);
    return '-';
  }
};

const validateNumber = (value) => {
  // 首先检查是否为有效输入
  if (value === null || value === undefined || value === '') {
    return false;
  }
  
  // 如果是数字类型，直接判断是否为有限数字
  if (typeof value === 'number') {
    return !isNaN(value) && isFinite(value);
  }
  
  // 如果是字符串，使用正则表达式验证是否为数字（可有两位小数）
  if (typeof value === 'string') {
    const regex = /^\d+(\.\d{1,2})?$/;
    return regex.test(value);
  }
  
  // 其他类型都返回false
  return false;
};






// totalReviewRatio 参数有需要上一个请求的total作为下一个请求的参数,才能获得本次请求的参数
watch([jcrSelectedOption, selectedOption, synthesisSelectedOption], () => {
  // 重置页码为第一页
  pageNum.value = 1;
  // 根据映射表转换为对应的数值
  jcr.value = jcrValueMap[jcrSelectedOption.value];
  // 根据分区映射表转换为对应的数值
  sub.value = subValueMap[selectedOption.value];
  // 根据综述映射表转换为对应的数值
  totalReviewRatio.value = synthesisValueMap[synthesisSelectedOption.value];
  // 数值正常输出,
  // console.log(totalReviewRatio.value);
  
  // 每次任一变量变化时，立即重新获取数据
  fetchJournalData();
});

// 2. 排序条件的监听，添加immediate: false选项
watch([order, orderType], () => {
  // 重置页码为第一页
  pageNum.value = 1;
  fetchJournalData();
}); // 这将防止组件创建时立即执行

// 3.影响因子输入框的值监听
watch([ifStart_2019, ifEnd_2019], () => {
  pageNum.value = 1;
  // 如果两个框都不为空，执行验证
  if (ifStart_2019.value !== '' && ifEnd_2019.value !== '') {
    // 验证开始值
    const isStartValid = validateNumber(ifStart_2019.value);
    // 验证结束值
    const isEndValid = validateNumber(ifEnd_2019.value);
    
    if (!isStartValid) {
      ElMessage.warning('请输入正确的开始影响因子值，最多两位小数');
      return;
    }
    
    if (!isEndValid) {
      ElMessage.warning('请输入正确的结束影响因子值，最多两位小数');
      return;
    }
    
    // 两个值都合法，获取数据
    fetchJournalData();
  }
});


// 添加全局点击事件监听器
onMounted(() => {
  document.addEventListener('click', closeDropdownOnOutsideClick);
  fetchJournalData();
});

// 组件卸载时移除事件监听器
onUnmounted(() => {
  document.removeEventListener('click', closeDropdownOnOutsideClick);
  // 移除输入框事件监听
  const startInput = document.querySelector('.filter-item input:first-of-type');
  const endInput = document.querySelector('.filter-item input:last-of-type');
  
  if (startInput) {
    startInput.removeEventListener('input', () => {});
    startInput.removeEventListener('blur', () => {});
  }
  
  if (endInput) {
    endInput.removeEventListener('input', () => {});
    endInput.removeEventListener('blur', () => {});
  }
});
</script>

<style scoped>
.filter-input.error {
  border-color: #F56C6C;
}

table {
  width: 100%;
  table-layout: fixed; /* 使用固定表格布局，这样可以更好地控制列宽 */
  border-collapse: collapse;
}

/* 设置第一列(期刊名称)宽度占比 */
th:first-child {
  width: 20%; /* 给期刊名称分配20%的表格宽度 */
}

/* 设置其他列平均分配剩余宽度 */
th:not(:first-child) {
  width: calc(80% / 9); /* 剩余9列平均分配80%的宽度 */
}

.hover-text {
  cursor: pointer;
  position: relative; /* 添加相对定位 */
  padding-right: 10px; /* 为图标留出空间 */
  white-space: nowrap; /* 防止文字换行 */
  overflow: hidden;
  text-overflow: ellipsis; /* 文字过长时显示省略号 */
}

.hover-text .sort-icon {
  position: absolute; /* 使用绝对定位替代浮动 */
  right: 5px; /* 距离右侧5px */
  top: 50%; /* 垂直居中 */
  transform: translateY(-50%); /* 完全居中 */
}

/* 鼠标悬停效果 */
.hover-text:hover {
  color: #00ACC1;
}



/* 鼠标悬停效果 */
.hover-text:hover {
  color: #00ACC1;
  cursor: pointer;
}

.nav-item:hover {
  color: #00ACC1;
}


/* header */
  .header {
  width: 100%;
  background-color: white;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  padding: 10px 0;
  }

  .header-content {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  }

  .logo-container {
  display: flex;
  align-items: center;
  }

  .logo-todedit {
  color: #0078C1;
  font-size: 28px;
  font-weight: bold;
  margin-right: 5px;
  }

  .logo-ijournal {
  color: #0090c1;
  font-size: 28px;
  font-weight: normal;
  font-style: italic;
  }

  .nav-menu {
  display: flex;
  flex-grow: 1;
  justify-content: center;
  gap: 20px;
  }

  .nav-item {
  padding: 10px 15px;
  font-size: 16px;
  color: #333;
  text-decoration: none;
  white-space: nowrap;
  }

  .login-btn {
  background-color: #00ACC1;
  color: white;
  padding: 8px 15px;
  border: none;
  border-radius: 3px;
  font-size: 16px;
  cursor: pointer;
  white-space: nowrap;
  }

/* 搜索区域样式 */
.search-section {
    width: 100%;
    background-color: #fff;
    border-bottom: 1px solid #eee;
}

.search-container {
    max-width: 1200px;
    width: 70%; /* 添加这行来缩小整体宽度 */
    margin: 0 auto;
    padding: 20px 20px 0 20px;
}

.search-box {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
    width: 800px;
    margin:0 auto;
}

.search-type {
    width: 120px;
    padding: 12px 15px;
    border-right: none;
    background-color: white;
    font-size: 14px;
    border-radius: 4px 0 0 4px;
    outline: none;
    cursor: pointer;
    appearance: none;
    background-image: url("data:image/svg+xml;charset=US-ASCII,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%22292.4%22%20height%3D%22292.4%22%3E%3Cpath%20fill%3D%22%23131313%22%20d%3D%22M287%2069.4a17.6%2017.6%200%200%200-13-5.4H18.4c-5%200-9.3%201.8-12.9%205.4A17.6%2017.6%200%200%200%200%2082.2c0%205%201.8%209.3%205.4%2012.9l128%20127.9c3.6%203.6%207.8%205.4%2012.8%205.4s9.2-1.8%2012.8-5.4L287%2095c3.5-3.5%205.4-7.8%205.4-12.8%200-5-1.9-9.2-5.5-12.8z%22%2F%3E%3C%2Fsvg%3E");
    background-repeat: no-repeat;
    background-position: right 12px top 50%;
    background-size: 10px auto;
    border: 1px solid #00ACC1; /* 修改边框颜色 */
}

.search-input {
    flex-grow: 1;
    padding: 12px 15px;
    font-size: 14px;
    outline: none;
    border: 1px solid #00ACC1; /* 修改边框颜色 */
}

.search-button {
    width: 60px;
    height: 45px;
    background-color: #00ACC1;
    color: white;
    border: none;
    border-radius: 0 4px 4px 0;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
}

.search-button i {
    font-size: 20px;
}

/* 过滤选项样式 */
.filter-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 15px 20px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    background-color: #fff;
}

.filter-item {
    display: flex;
    align-items: center;
    margin-right: 20px;
}

.filter-label {
    font-size: 14px;
    color: #333;
    margin-right: 10px;
}

.filter-input {
    width: 70px;
    height: 30px;
    padding: 5px 8px;
    border: 1px solid #ddd;
    border-radius: 2px;
    outline: none;
}

.filter-select {
    display: flex;
    align-items: center;
    font-size: 14px;
    color: #666;
    cursor: pointer;
}

.dropdown {
  display: inline-flex;
  align-items: center;
  cursor: pointer;
  border: none !important;
}

.dropdown:after {
    content: "▼";
    font-size: 10px;
    margin-left: 5px;
    color: #999;
}

/* 分隔线 */
.separator {
    display: inline-block;
    width: 10px;
    text-align: center;
    color: #ddd;
}


/* 表格区域样式 - 修改匹配图片 */
.journal-table-container {
  max-width: 1200px;
  margin: 20px auto;
  background-color: white;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  padding: 20px;
}


.journal-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}

.journal-table th {
  background-color: #fff;
  padding: 12px 18px;
  text-align: center;
  font-weight: 400;
  border-bottom: 1px solid #ddd;
  color: #333;
}

.journal-table td {
  padding: 12px 8px;
  text-align: center;
  border-bottom: 1px solid #fff;
  color: #333;
}

.journal-table tr:hover {
  background-color: #f5f5f5;
}

.journal-name {
  text-align: left;
  font-weight: bold;
  color: #333;
}

.journal-abbr {
  text-align: center;
  color: #666;
  font-size: 12px;
  display: block;
  margin-top: 3px;
}

.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  align-items: center;
}

.pagination a {
  padding: 5px 10px;
  margin: 0 3px;
  border: 1px solid #ddd;
  color: #666;
  text-decoration: none;
  border-radius: 3px;
  font-size: 14px;
}

.pagination .active {
  background-color: #1e90ff;
  color: white;
  border-color: #1e90ff;
}

.pagination-goto {
  margin-left: 15px;
  color: #666;
  font-size: 14px;
  display: flex;
  align-items: center;
}

.pagination-goto input {
  width: 40px;
  height: 26px;
  margin: 0 5px;
  text-align: center;
  border: 1px solid #ddd;
  border-radius: 3px;
}

/* 页脚样式 */
.footer {
  width: 100%;
  height: 250px;
  background-color: #242730;
  color: white;
  padding: 40px 0 20px 0;
  margin-top: 40px;
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  padding: 0 20px;
}



.footer-columns {
  display: flex;
  flex-wrap: wrap;
  flex-grow: 1;
  justify-content: space-between;
}

.footer-column {
  margin-right: 30px;
  min-width: 120px;
}

.footer-column h3 {
  font-size: 16px;
  font-weight: normal;
  margin-bottom: 15px;
  color: white;
}

.footer-link {
  display: block;
  color: #aaa;
  font-size: 14px;
  text-decoration: none;
  margin-bottom: 10px;
  transition: color 0.3s;
}

.footer-link:hover {
  color: white;
}

.footer-bottom {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  max-width: 1200px;
  margin: 0px auto 0;
  padding: 20px 20px 0;
}

.copyright {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  color: #aaa;  
  font-size: 14px;
  width: 100%; /* 确保容器有足够的宽度 */
}

.copyright img{
  height: 30px;
  margin-left: -100px;
  margin-top: 10px;
}

/* footer 页脚文字处理 */
.logo-container {
  margin-top: 0;
  padding-right: 80px;
  display: flex;
  align-items: flex-start;
  justify-content: flex-start;
}

.todedit {
  font-family: 'Arial', sans-serif;
  font-weight: bold;
  font-size: 28px;
  color: white;
  letter-spacing: 1px;
  text-transform: uppercase;
}

.separator {
  color: white;
  margin: 0 15px;
  font-size: 28px;
  font-weight: 200;
}

.ijournal {
  font-family: 'Arial', sans-serif;
  font-size: 28px;
  color: white;
  font-weight: 200;
  letter-spacing: 1px;
}

.separator-corridor{
  text-decoration: black;
}

/* 下拉选择框的样式 */
.filter-item {
  position: relative;
  margin-bottom: 10px;
}

.filter-label {
  margin-right: 10px;
}

.dropdown {
  position: relative;
  display: inline-block;
  padding: 5px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  left: 0;
  z-index: 10;
  min-width: 100%;
  margin-top: 5px;
  background-color: white;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.dropdown-item {
  padding: 8px 12px;
  cursor: pointer;
}

.dropdown-item:hover {
  background-color: #f5f5f5;
}






</style>