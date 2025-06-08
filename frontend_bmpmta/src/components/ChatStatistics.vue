<script setup>
import * as echarts from 'echarts';
import { onMounted, ref, nextTick, onBeforeUnmount } from "vue";
// import { useRouter } from "vue-router";

// 使用 defineEmits 定义自定义事件，实现子组件与父组件之间的通信
const emit = defineEmits(['change-component']);

const chartContainer = ref(null);
let myChart = null;

// 预定义的节点数据
const nodeData = [
  // 中心节点
  { id: '0', name: '中心节点', category: 0, value: 80, symbolSize: 40, componentName: 'ThinkingPage' },

  // 分类0节点
  { id: '1', name: '节点1', category: 0, value: 70, symbolSize: 25 },
  { id: '2', name: '节点2', category: 0, value: 65, symbolSize: 20 },
  { id: '3', name: '节点3', category: 0, value: 85, symbolSize: 25 },

  // 分类1节点
  { id: '4', name: '节点4', category: 1, value: 50, symbolSize: 18 },
  { id: '5', name: '节点5', category: 1, value: 45, symbolSize: 15 },
  { id: '6', name: '节点6', category: 1, value: 55, symbolSize: 20 },
  { id: '7', name: '节点7', category: 1, value: 60, symbolSize: 22 },

  // 分类2节点
  { id: '8', name: '节点8', category: 2, value: 30, symbolSize: 15 },
  { id: '9', name: '节点9', category: 2, value: 35, symbolSize: 18 },
  { id: '10', name: '节点10', category: 2, value: 25, symbolSize: 12 },

  // 分类3节点
  { id: '11', name: '节点11', category: 3, value: 90, symbolSize: 28 },
  { id: '12', name: '节点12', category: 3, value: 95, symbolSize: 30 },

  // 分类4节点
  { id: '13', name: '节点13', category: 4, value: 10, symbolSize: 10 },
  { id: '14', name: '节点14', category: 4, value: 15, symbolSize: 12 },
  { id: '15', name: '节点15', category: 4, value: 5, symbolSize: 8 },
  { id: '16', name: '节点16', category: 4, value: 20, symbolSize: 14 },

  // 跨类连接节点
  { id: '17', name: '桥接1', category: 0, value: 55, symbolSize: 22 }, // 连接类别0和1
  { id: '18', name: '桥接2', category: 1, value: 32, symbolSize: 18 }, // 连接类别1和2
  { id: '19', name: '桥接3', category: 2, value: 88, symbolSize: 25 }, // 连接类别2和3
  { id: '20', name: '桥接4', category: 3, value: 12, symbolSize: 15 }  // 连接类别3和4
];

// 基于节点属性决定是否应该连接
const shouldConnect = (sourceNode, targetNode) => {
  // 连接策略：
  // 1. 同类别节点相连 (category 相同)
  // 2. 或者 value 差值小于 20 的节点相连
  return (
      sourceNode.category === targetNode.category ||
      Math.abs(sourceNode.value - targetNode.value) < 20
  );
};

// 初始化图表的函数
const initChart = () => {
  if (!chartContainer.value) return;

  // 销毁旧的实例
  if (myChart) {
    myChart.dispose();
  }

  console.log('初始化图表');

  // 初始化图表
  myChart = echarts.init(chartContainer.value);

  // 设置颜色映射，用于不同类别的节点
  const colors = ['#c23531', '#2f4554', '#61a0a8', '#d48265', '#91c7ae'];

  // 为节点添加颜色和初始位置
  const nodes = nodeData.map((node, index) => {
    // 为每个节点添加初始位置
    // 使用圆形布局放置节点
    const angle = (index / nodeData.length) * Math.PI * 2;
    const radius = 150; // 圆的半径

    return {
      ...node,
      // 中心节点放在中间，其他节点围绕它
      x: index === 0 ? 0 : Math.cos(angle) * radius,
      y: index === 0 ? 0 : Math.sin(angle) * radius,
      fixed: index === 0, // 只固定中心节点
      itemStyle: {
        color: colors[node.category]
      }
    };
  });

  // 构建连接
  const edges = [];
  for (let i = 0; i < nodes.length; i++) {
    for (let j = i + 1; j < nodes.length; j++) {
      if (shouldConnect(nodes[i], nodes[j])) {
        edges.push({
          source: nodes[i].id,
          target: nodes[j].id,
          lineStyle: {
            width: 1 + Math.random() * 2, // 线宽随机变化
            color: colors[nodes[i].category] // 使用源节点的颜色
          }
        });
      }
    }
  }

  // 设置图表配置
  const nodeOption = {
    title: {
      text: '基于节点属性的连接',
      subtext: '同类别或数值相近的节点相连',
      left: 'center'
    },
    tooltip: {
      // 定义提示内容
      formatter: function(params) {
        if (params.dataType === 'node') {
          return `节点: ${params.data.name}<br/>` +
              `类别: ${params.data.category}<br/>` +
              `数值: ${params.data.value}`;
        } else {
          return `连接: ${params.data.source} ⟷ ${params.data.target}`;
        }
      }
    },
    legend: [{
      data: ['分类0', '分类1', '分类2', '分类3', '分类4'],
      left: 'left',
      orient: 'vertical',
      selected: {
        '分类0': true,
        '分类1': true,
        '分类2': true,
        '分类3': true,
        '分类4': true
      }
    }],
    toolbox: {
      show: true,
      feature: {
        saveAsImage: {},
        restore: {},
        dataView: { readOnly: true },
        brush: {},
        dataZoom: {}
      }
    },
    series: [
      {
        type: 'graph',
        layout: 'force',
        animation: true,
        data: nodes,
        categories: [
          { name: '分类0' },
          { name: '分类1' },
          { name: '分类2' },
          { name: '分类3' },
          { name: '分类4' }
        ],
        force: {
          initLayout: 'circular', // 强制使用圆形初始布局
          repulsion: 200,
          edgeLength: 80,
          gravity: 0.1
        },
        edges: edges,
        roam: true,
        label: {
          show: true,
          position: 'right',
          formatter: '{b}'
        },
        emphasis: {
          focus: 'adjacency',
          lineStyle: {
            width: 3
          },
          scale: true
        },
        universalTransition: true
      }
    ]
  };

  const barOption = {
    xAxis: {
      type: 'value'
    },
    yAxis: {
      type: 'category',
      axisLabel: {
        rotate: 30
      },
      data: nodeData.map(function (item) {
        return item.name;
      })
    },
    animationDurationUpdate: 1000,
    series: {
      type: 'bar',
      id: 'population',
      data: nodeData.map(function (item) {
        return item.value;
      }),
      universalTransition: true
    }
  };

  // 应用配置
  myChart.setOption(nodeOption);

  // 设置默认视图为节点视图
  let currentOption = nodeOption

  // 执行两种数据视图的切换
  setInterval(function () {
    currentOption = currentOption === nodeOption ? barOption : nodeOption;
    myChart.setOption(currentOption, true);
  }, 5000);

  // 稍后再次调整大小以确保显示正确
  setTimeout(() => {
    if (myChart) {
      myChart.resize();
    }
  }, 200);

  console.log('图表初始化完成');

  // 添加点击事件
  myChart.on('click', function(params) {
    if (params.dataType === 'node') {
      console.log('点击了节点:', params.data.name);
      // 这里可以添加导航逻辑
      // router.push(params.data.url); // 跳转到指定路由
      // 将
      if (params.dataType === 'node' && params.data.componentName) {
        emit('change-component', params.data.componentName);
      }
    }
  });
};


onMounted(() => {
  console.log('组件已挂载');

  // 使用nextTick确保DOM已渲染
  nextTick(() => {
    // 延迟初始化以确保DOM已完全渲染
    setTimeout(() => {
      initChart();
    }, 0);
  });

  // 处理窗口大小变化
  const handleResize = () => {
    if (myChart) myChart.resize();
  };

  window.addEventListener('resize', handleResize);

  // 在组件销毁前，执行清理操作
  onBeforeUnmount(() => {
    window.removeEventListener('resize', handleResize);
    if (myChart) myChart.dispose();
  });
});
</script>

<template>
  <div class="chart-container">
    <div class="chart-wrapper">
      <div ref="chartContainer" class="chart-area"></div>
    </div>
    <div class="controls">
      <div class="legend-info">
        <h3>连接规则说明</h3>
        <p>节点在以下情况下建立连接：</p>
        <ul>
          <li>属于相同类别的节点相互连接</li>
          <li>数值差异小于20的节点相互连接</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chart-container {
  width: 100%;
  max-width: 1000px;
  margin: 0 auto;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  border-radius: 4px;
  padding: 16px;
}

.chart-wrapper {
  border: 1px solid #eaeaea;
  border-radius: 4px;
  overflow: hidden;
  width: 100%;
  height: 600px;
}

.chart-area {
  width: 100%;
  height: 100%;
}

.controls {
  margin-top: 20px;
  padding: 10px;
  background-color: #f5f5f5;
  border-radius: 4px;
}

.legend-info {
  margin-bottom: 10px;
}

.legend-info h3 {
  font-size: 16px;
  margin-bottom: 8px;
}

.legend-info ul {
  padding-left: 20px;
}

.legend-info li {
  margin-bottom: 4px;
}
</style>