<template>
  <div class="root">
    <div
      id="headPanel"
      :class="{ hidden: headVisible }"
    >
      <span class="logo">脑图</span>
      <i
        class="gb-toggle-btn"
        @click="headVisible = !headVisible"
      />
    </div>
    <!-- 左侧按钮 -->
    <item-panel
      :graph="graph"
      @canvas-add-node="addNode"
    />
    <!-- 挂载节点 -->
    <div
      id="canvasPanel"
      @dragover.prevent
    />
    <!-- 配置面板 -->
    <div
      id="configPanel"
      :class="{ hidden: !configVisible }"
    >
      <i
        class="gb-toggle-btn"
        @click="configVisible = !configVisible"
      />
      <h2 class="panel-title">数据配置</h2>
      <div class="config-data">
        id: {{ config.id }}, data: {{ config.data }}
      </div>
      <h2 class="panel-title">节点样式配置</h2>
      <div class="config-data">
        <div class="config-item">
          形状: <select>
            <option
              v-for="(item, index) in nodeShapes"
              :key="index"
              :value="item.shape"
            >
              {{ item.name }}
            </option>
          </select>
        </div>
        <div class="config-item">
          背景色: <input v-model="node.fill">
        </div>
        <div class="config-item">
          边框虚线: <input v-model="node.lineDash">
        </div>
        <div class="config-item">
          边框颜色: <input v-model="node.borderColor">
        </div>
        <div class="config-item">
          宽: <input v-model="node.width">px
        </div>
        <div class="config-item">
          高: <input v-model="node.height">px
        </div>
      </div>
      <h2 class="panel-title">文字样式配置</h2>
      <div class="config-data">
        <div class="config-item">
          文字: <input v-model="label">
        </div>
        <div class="config-item">
          字体大小: <input v-model="labelCfg.fontSize">
        </div>
        <div class="config-item">
          颜色: <input v-model="labelCfg.fill">
        </div>
      </div>
      <button @click="configVisible = false">取消</button>
      <button
        class="save"
        @click="save"
      >
        保存
      </button>
    </div>
    <div
      v-if="tooltip"
      class="g6-tooltip"
      :style="`top: ${top}px; left: ${left}px;`"
    >
      id: {{ tooltip }}
    </div>
  </div>
</template>

<script>
import G6 from '../../components/graph/graph';
import ItemPanel from './ItemPanel.vue';

export default {
  components: {
    ItemPanel,
  },
  data () {
    return {
      graph:     {},
      highLight: {
        undo: false,
        redo: false,
      },
      // 保存线条样式
      lineStyle: {
        type:  'line',
        width: 1,
      },
      label:    '',
      labelCfg: {
        fontSize: 12,
        fill:     '#fff',
      },
      node: {
        fill:        '',
        lineDash:    'none',
        borderColor: '',
        width:       160,
        height:      60,
        shape:       'rect-node',
      },
      nodeShapes: [
        {
          name:  '矩形',
          shape: 'rect-node',
        },
        {
          name:  '圆形',
          shape: 'circle-node',
        },
        {
          name:  '椭圆',
          shape: 'ellipise-node',
        },
        {
          name:  '菱形',
          shape: 'diamond-node',
        },
      ],
      headVisible:   false,
      configVisible: false,
      config:        '',
      tooltip:       '',
      top:           0,
      left:          0,
      canvasOffset:  {
        x: 0,
        y: 0,
      },
      selectedItem: null,
    };
  },
  mounted () {
    document.title = '脑图';
    // 创建画布
    this.$nextTick(() => {
      this.createGraphic();
      this.initGraphEvent();
    });
  },
  beforeDestroy () {
    this.graph.destroy();
  },
  methods: {
    createGraphic () {
      const data = {
        id:        'root',
        label:     'Root',
        // 边的样式
        edgeStyle: {
          stroke: '#999',
        },
        children: [{
          data: {
            action: '初始化',
          },
          // note:      'root-note',
          type:      'modelRect-node',
          edgeStyle: { // 节点样式
            stroke: '#39495b',
          },
          // 左侧方条
          preRect: {
            show:   true, // 是否显示左侧方条
            width:  4,
            fill:   '#40a9ff',
            radius: 2,
          },
          logoIcon: {
            show:   true, // 是否显示图标
            x:      -92,  // 控制图标在横轴上的位置
            y:      -8,   // 控制图标在纵轴上的位置
            img:    'https://gw.alipayobjects.com/zos/basement_prod/4f81893c-1806-4de4-aff3-9a6b266bc8a2.svg',
            width:  16,
            height: 16,
          },
          // 状态图标, 注意: 坐标 (0, 0) 代表图形几何中心
          stateIcon: {
            show:       true,     // 是否显示图标
            x:          70,       // 控制图标在横轴上的位置
            y:          10,       // 控制图标在纵轴上的位置
            text:       '\ue610',
            fontFamily: 'iconfont',
            fontSize:   20,
            style:      {
              stroke: '#333',
              fill:   '#fc0',
            },
          },
          labels: [{
            x:        -70,
            y:        -10,
            label:    '标题,最长10个字符~~',
            labelCfg: {
              fill:      '#666',
              fontSize:  14,
              maxlength: 10,
            },
          }, {
            x:        -70,
            y:        7,
            label:    '描述,最长12个字符~~~',
            labelCfg: {
              fontSize:  12,
              fill:      '#999',
              maxlength: 12,
            },
          }, {
            x:        -70,
            y:        24,
            label:    '第三行,最长16个字符,超出显示省略号~~~',
            labelCfg: {
              fontSize:  10,
              fill:      '#ccc',
              maxlength: 16,
            },
          }],
          children: [{
            label:    '初始化事件和生命周期和其他',
            labelCfg: {
              stroke:    '#ccc',
              fill:      '#666',
              maxlength: 10,
            },
          }],
        }, {
          label:    '初始化事件和生命周期和其他',
          labelCfg: {
            stroke:    '#ccc',
            fill:      '#666',
            maxlength: 10,
          },
          // 状态图标, 注意: 坐标 (0, 0) 代表图形几何中心
          stateIcon: {
            show:       true,     // 是否显示图标
            x:          -96,      // 控制图标在横轴上的位置
            y:          14,       // 控制图标在纵轴上的位置
            text:       '\ue601',
            fontFamily: 'iconfont',
            fontSize:   30,
            style:      {
              stroke: '#999',
              fill:   '#fc0',
            },
          },
          children: [{
            label:    '初始化事件和生命周期和其他',
            labelCfg: {
              stroke:    '#ccc',
              fill:      '#666',
              maxlength: 10,
            },
          }],
        }],
      };

      for(let i = 0; i < 11; i++) {
        data.children.push({
          label:    '初始化事件和生命周期和其他',
          labelCfg: {
            stroke:    '#ccc',
            fill:      '#666',
            maxlength: 10,
          },
          children: [{
            label:    '初始化事件和生命周期和其他',
            labelCfg: {
              stroke:    '#ccc',
              fill:      '#666',
              maxlength: 10,
            },
          }],
        });
      }
      const graph = new G6({
        width:     window.innerWidth - 40,
        height:    window.innerHeight - 40,
        fitCenter: true,
        layout:    {
          type:      'mindmap',
          direction: 'H', // H / V / LR / RL / TB / BT
          getWidth:  () => {
            return 200;
          },
          getVGap: () => {
            return 50;
          },
          getHGap: () => {
            return 50;
          },
        },
        modes: {
          default: ['zoom-canvas', 'drag-canvas', 'canvas-event'],
        },
        // renderer:    'svg',
        defaultNode: {
          type:     'modelRect-node',
          labelCfg: {
            fontSize: 20,
          },
          // 锚点控制字段
          anchorControls: {
            hide: true,
          },
        },
        defaultEdge: {
          type:  'hvh-h-edge', // 扩展了内置边, 有边的事件
          style: {
            radius:          5,
            offset:          15,
            lineWidth:       3,
            lineAppendWidth: 10, // 防止线太细没法点中
          },
        },
        // 覆盖全局样式
        nodeStateStyles: {
          'nodeState:default': {
            opacity: 1,
          },
          'nodeState:hover': {
            opacity: 0.8,
          },
          'nodeState:selected': {
            opacity: 0.9,
          },
        },
        // 默认边不同状态下的样式集合
        edgeStateStyles: {
          'edgeState:hover': {
            animate: false,
          },
        },
        // 自定义注册行为, 事件, 交互
        registerFactory: (G6, cfg) => {
          const minimap = new G6.Minimap({
            size: [200, 100],
          });

          cfg.plugins = [minimap];

          return new G6.TreeGraph(cfg);
        },
        // ... 其他G6原生入参
      });

      this.graph = graph.instance;
      this.graph.read(data);
      setTimeout(() => {
        this.graph.paint();
        this.graph.fitView();
      });
    },
    // 添加节点
    addNode (e) {
      const model = {
        text: 'node',
        // id:  Util.uniqueId(),
        // 形状
        type: e.target.dataset.shape,
        // 坐标
        x:    e.clientX + this.canvasOffset.x - 80,
        y:    e.clientY + this.canvasOffset.y - 40,
      };

      this.graph.addItem('node', model);
    },
    // 初始化图事件
    initGraphEvent () {
      this.graph.on('on-canvas-dragend', e => {
        this.canvasOffset.x = e.dx;
        this.canvasOffset.y = e.dy;
      });

      this.graph.on('after-node-selected', e => {
        this.configVisible = !!e;

        if (e && e.item) {
          const model = e.item.get('model');

          this.selectedItem = e.item;
          this.config = model;
          this.label = model.label;
          this.labelCfg = {
            fill:     model.labelCfg.fill,
            fontSize: model.labelCfg.fontSize,
          };
          this.node = {
            fill:        model.style.fill,
            borderColor: model.style.stroke,
            lineDash:    model.style.lineDash || 'none',
            width:       model.style.width,
            height:      model.style.height,
            shape:       model.type,
          };

          // model.label = e.item.get('id');
          /* this.graph.updateItem(e.item, {
            x: 100,
            y: 100,
          }); */
        }
      });

      this.graph.on('after-edge-selected', e => {
        this.configVisible = !!e;

        if (e && e.item) {
          this.config = e.item.get('model').id;

          this.graph.updateItem(e.item, {
            // shape: 'line-edge',
            style: {
              radius:    10,
              lineWidth: 2,
            },
          });
        }
      });

      this.graph.on('on-edge-mousemove', e => {
        if (e && e.item) {
          this.tooltip = e.item.get('model').label;
          this.left = e.clientX + 40;
          this.top = e.clientY - 20;
        }
      });

      this.graph.on('before-node-removed', ({ target, callback }) => {
        console.log(target);
        setTimeout(() => {
          callback(true);
        }, 100);
      });

      this.graph.on('after-node-dblclick', e => {
        if (e && e.item) {
          console.log(e.item);
        }
      });

      this.graph.on('before-edge-add', ({ source, target, sourceAnchor, targetAnchor }) => {
        setTimeout(() => {
          this.graph.addItem('edge', {
            source: source.get('id'),
            target: target.get('id'),
            sourceAnchor,
            targetAnchor,
          });
        }, 100);
      });
    },
    save() {
      this.graph.updateItem(this.selectedItem, {
        label: this.label,
      });
      this.configVisible = false;
    },
  },
};
</script>

<style lang="scss">
  /* 提示框的样式 */
  .g6-tooltip {
    position: fixed;
    top: 0;
    left: 0;
    font-size: 12px;
    color: #545454;
    border-radius: 4px;
    border: 1px solid #e2e2e2;
    background-color: rgba(255, 255, 255, 0.9);
    box-shadow: rgb(174, 174, 174) 0 0 10px;
    padding: 10px 8px;
  }
  .g6-minimap{
    position: absolute;
    right: 0;
    bottom: 0;
  }
</style>
