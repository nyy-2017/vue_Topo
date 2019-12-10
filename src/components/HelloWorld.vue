<template>
  <div>
    <Row :gutter="24">
      <Col span="5" style="display:flex;">
        <Select v-model="value" label-in-value clearable @on-change="addNode">
          <Option v-for="item in types" :label="item.label" :value="item.value" :key="item.value">
            <img style="height:20px;width:20px;" :src="item.value" />
            <span>{{ item.label }}</span>
          </Option>
        </Select>
      </Col>
      <!-- <Button type="primary" @click="addNode">添加</Button> -->
      <Col span="3">
      <!-- 默认模式和框选模式 -->
        <ul class="svgHeadItemLst">
          <li class="svgHeadItem" v-for="(ele,key) in svgToolbar" :key="ele.className" :class="{'active':ele.isActive}" @mousedown="selectToolbar(key)"  :title="ele.name">
            <div class="svgHeadItemImg" :class="ele.className"></div>
          </li>
        </ul>
      </Col>
      <Col span="3">
        <Button type="primary" @click="editTopo">编辑</Button>
      </Col>
      <Col span="3">
        <Button id="save_json" type="primary" ghost @click="saveTopoJson">保存Json文件</Button>
      </Col>
      <Col span="3">
        <Button type="primary" @click="saveAsPng">下载为图片</Button>
      </Col>
      <Col span="3">
        <Button type="primary" @click="open_changeFile">打开json文件</Button>
      </Col>
    </Row>
    <div class="svgMain">
      <!-- 左侧的图片库 -->
      <v-shapebar @click="dragShapeNode"></v-shapebar>
      <!-- 画布layout -->
      <div :id="'topoId'+topoId" class="topoWrap">
        <svg
          class="topoSvg"
          :width="svgAttr.width"
          :height="svgAttr.height"
          :viewBox="svgAttr.viewX+' '+svgAttr.viewY+' '+svgAttr.width+' '+svgAttr.height"
          @mousedown.stop="mousedownTopoSvg($event)">
          <!-- @mousewheel="mouseWheel($event)" -->
          <defs>
            <pattern id="Pattern" x="0" y="0" width="100" height="100" patternUnits="userSpaceOnUse">
              <line :x1="ele.x1" :x2="ele.x2" :y1="ele.y1" :y2="ele.y2" :stroke="ele.color" :stroke-width="ele.strokeWidth" :opacity="ele.opacity" v-for="ele in gridData" :key="ele.id"/>
            </pattern>
          </defs>
          <defs id="SvgjsDefs1370">
            <marker id="arrow" markerWidth="12" markerHeight="12" refX="6" refY="6" viewBox="0 0 12 12" orient="auto">
              <path id="SvgjsPath1372" d="M2,2 L2,9 L8,5 L2,2" stroke="none" fill="#666" />
            </marker>
          </defs>
          <defs>
            <filter id="f1" x="0" y="0" width="200%" height="200%">
              <feOffset result="offOut" in="SourceGraphic" dx="2" dy="2" />
              <feColorMatrix result="matrixOut" in="offOut" type="matrix" values="0.2 0 0 0 0 0 0.2 0 0 0 0 0 0.2 0 0 0 0 0 1 0"/>
              <feGaussianBlur result="blurOut" in="matrixOut" stdDeviation="2" />
              <feBlend in="SourceGraphic" in2="blurOut" mode="normal" />
            </filter>
          </defs>
          <rect v-if="isEdit" fill="url(#Pattern)" :width="svgAttr.width" :height="svgAttr.height" />
          <g :transform="scale">
            <!-- 节点 图片-->
            <g
              class="nodesG"
              v-for="(ele,key) in nodes"
              :key="ele.id"
              @mousedown.stop="dragSvgNode(key,$event)"
              @mouseover.stop="mouseoverNode(key,$event)"
              @mouseout.stop="mouseoutNode(key,$event)"
              :transform="'translate('+ele.x+','+ele.y+')'">
              <image class="nodeImage" :xlink:href="ele.icon" :width="ele.width" :height="ele.height" :x="0" :y="0" :href="ele.icon"/>
              <!-- <text class="nodeName" :x="10" :y="ele.height + 20" text-anchor='start' >{{ele.name}}</text> -->
              <!-- <text class="nodeName" :x="10" :y="ele.height + 14" text-anchor='start' >{{ele.name}}</text> -->
              <text class="nodeName" :x="-50" :y="ele.height -20" text-anchor="start">{{ele.name}}</text>

              <rect class="reactClass" v-show="ele.isTopConnectShow" :x="-2" :y="-1" :rx="0" :ry="0" :width="ele.width + 4" :height="ele.height + 2"/>
              <g class="connectorArror" :class="{'connector':ele.isTopConnectShow}" :transform="'translate(' + ele.width/2 +','+ 0 +')'" v-show="ele.isTopConnectShow">
                <circle r="10" cx="0" cy="0" class="circleColor" fill="#efefef" />
                <!-- <line x1="-4" y1="1" x2="0" y2="-5" stroke="#333" stroke-width="1"></line>
                <line x1="4" y1="1" x2="0" y2="-5" stroke="#333" stroke-width="1"></line>-->
                <line x1="-4" y1="-1" x2="0" y2="-5" stroke="#333" />
                <line x1="4" y1="-1" x2="0" y2="-5" stroke="#333" />
              </g>
              <!-- X号的：2条线拼接长 -->
              <g
                class="connectorArror"
                v-show="ele.isBottomConnectShow"
                :class="{'connector':ele.isBottomConnectShow}"
                :transform="'translate('+ele.width/2+','+ele.height+')'"
                @mousedown.stop="drawConnectLine(key,$event)"
                @mouseout.stop="mouseoutLeftConnector(key)">
                <circle r="10" cx="0" cy="0" class="circleColor" fill="#efefef" />
                <line x1="-4" y1="-1" x2="0" y2="5" stroke="#333" />
                <line x1="4" y1="-1" x2="0" y2="5" stroke="#333" />
              </g>
              <!-- 删除节点 -->
              <g
                class="deleteArror"
                :class="{'connector':ele.isTopConnectShow}"
                :transform="'translate(' + ele.width +','+ 0 +')'"
                @mousedown.stop="deleteNode(key,$event)"
                v-show="ele.isTopConnectShow">
                <circle r="10" cx="0" cy="-0" class="circleColor" fill="#efefef" />
                <line x1="-3" y1="-3" x2="3" y2="3" stroke="#333" stroke-width="1" />
                <line x1="-3" y1="3" x2="3" y2="-3" stroke="#333" stroke-width="1" />
              </g>
            </g>
            <!-- 连线 -->
            <g
              class="connectorsG"
              v-for="(ele,key) in connectors"
              :class="{active:ele.isSelect}"
              :key="ele.id"
              v-if="ele.type == 'Line'"
              @mouseover.stop="selectConnectorLine(key)"
              @mouseout.stop="cancelSelectConnectorLine(key)">
              <!-- 连线方式一 -->
              <path
                class="connectorLine"
                :stroke="ele.color"
                :stroke-width="ele.strokeW"
                marker-end="url(#arrow)"
                :d="'M'+(ele.targetNode.x + ele.targetNode.width /2)+','+(ele.targetNode.y -10) +
                'L'+(ele.sourceNode.x + ele.sourceNode.width / 2)+','+(ele.sourceNode.y + ele.sourceNode.height + 10)+
                'Z'"/>
              <!-- 删除连线 -->
              <g
                class="deleteArror"
                :class="{'connector':ele.isTopConnectShow}"
                :transform="'translate(' + (ele.targetNode.x + ele.targetNode.width /2 + ele.sourceNode.x + ele.sourceNode.width / 2) / 2 +','
                + (ele.targetNode.y -10 + ele.sourceNode.y + ele.sourceNode.height + 10) / 2 +')'"
                @mousedown.stop="deleteLine(key,$event)"
                v-show="ele.showDelete">
                <circle r="12" cx="0" cy="-0" class="circleColor" fill="#efefef" />
                <line x1="-3" y1="-3" x2="3" y2="3" stroke="#333" stroke-width="1" />
                <line x1="-3" y1="3" x2="3" y2="-3" stroke="#333" stroke-width="1" />
              </g>
            </g>
            <!-- 动态绘制的连线 -->
            <g>
              <line :x1="connectingLine.x1" :y1="connectingLine.y1" :x2="connectingLine.x2" :y2="connectingLine.y2" v-show="connectingLine.isConnecting" marker-end="url(#arrow)" stroke="#666" stroke-width="1"/>
            </g>
          </g>
        </svg>
      </div>
    </div>
    <!-- 拖拽左侧图片，显示拖拽的图片 -->
    <div class="moveNode nodeMoveCss" v-if="shapebarMoveNode.isShow" :style="{ left:shapebarMoveNode.left + 'px', top: shapebarMoveNode.top + 'px' }">
      <div class="shapeIcon">
        <img class="shapeIconImg" :src="shapebarMoveNode.icon"/>
      </div>
      <div class="shapeName">{{shapebarMoveNode.name}}</div>
    </div>
  </div>
</template>

<script>
import vShapebar from "./TopoModal/vShapebar";
// import img0 from '../../static/img/svg/SB-0.png';
// import img1 from '../../static/img/svg/FWQ-0.png';
// import img2 from '../../static/img/svg/JHJ-0.png';
// import img3 from '../../static/img/svg/IP-0.png';
// import img4 from '../../static/img/svg/YSQ-0.png';
// status = 0 是默认选中时候的照片， 1 绿色图片，2红色图片
export default {
  components: { "v-shapebar": vShapebar },
  data() {
    return {
        svgToolbar:[
            {name:'默认模式',className:'toolbar-default',isActive:true},
            {name:'框选模式',className:'toolbar-rectangle_selection',isActive:false},
            // {name:'放大',className:'toolbar-zoomin',isActive:false},
            // {name:'缩小',className:'toolbar-zoomout',isActive:false},
            // {name:'恢复出厂设置',className:'toolbar-zoomreset',isActive:false}
        ],
        // 拖拽时候的 小图标的初始化
        shapebarMoveNode:{
          left:0,
          top:0,
          name:'',
          icon:'',
          isShow:false
        },
        value: null,
        isEdit: false, // 编辑开关默认(关闭)
        scale: "scale(1)", // 画布(放大/缩小)初始值
        rate: 1, // 画布(放大/缩小)初始值
        types: [
            { label: "设备", value: require("../../static/img/svg/SB-0.png") },
            { label: "IP", value: require("../../static/img/svg/IP-0.png") },
            { label: "服务器", value: require("../../static/img/svg/FWQ-0.png") },
            { label: "交换机", value: require("../../static/img/svg/JHJ-0.png") },
            { label: "扬声器", value: require("../../static/img/svg/YSQ-0.png") }
        ],
        svgAttr: {
            width: 1410,
            height: 700,
            viewX: 0,
            viewY: 0,
            // minW: 800,
            // minH: 700
        },
        marker: {
            xmarkerY: 0,
            xmarkerX: 0,
            ymarkerX: 0,
            ymarkerY: 0,
            isMarkerShow: false
        },
        connectingLine: {
            x1: 0,
            y1: 0,
            x2: 0,
            y2: 0,
            isConnecting: true,
            sourceNode: "",
            endNode: ""
        },
        topoId: "",
        selectNodeData: {},
        // 结点
        nodes: [
            {
            name: "设备",
            id: 1,
            x: 300,
            y: 20,
            status: 0,
            icon: require("../../static/img/svg/SB-0.png"),
            width: 50,
            height: 50,
            containNodes: [2, 3, 4],
            attrs: [],
            classType: "T2",
            isTopConnectShow: false,
            isBottomConnectShow: false
            },
            {
            name: "交换机",
            id: 2,
            x: 50,
            y: 150,
            status: 0,
            icon: require("../../static/img/svg/JHJ-0.png"),
            width: 50,
            height: 50,
            containNodes: [],
            attrs: [],
            classType: "T2",
            isTopConnectShow: false,
            isBottomConnectShow: false
            },
            {
            name: "IP",
            id: 3,
            x: 200,
            y: 150,
            status: 0,
            icon: require("../../static/img/svg/IP-0.png"),
            width: 50,
            height: 50,
            containNodes: [],
            attrs: [],
            classType: "T2",
            isTopConnectShow: false,
            isBottomConnectShow: false
            },
            {
            name: "扬声器",
            id: 4,
            x: 400,
            y: 150,
            status: 0,
            icon: require("../../static/img/svg/YSQ-0.png"),
            width: 50,
            height: 50,
            containNodes: [5],
            attrs: [],
            classType: "T2",
            isTopConnectShow: false,
            isBottomConnectShow: false
            },
            {
            name: "扬声器2",
            id: 5,
            x: 200,
            y: 300,
            status: 0,
            icon: require("../../static/img/svg/YSQ-0.png"),
            width: 50,
            height: 50,
            containNodes: [],
            attrs: [],
            classType: "T2",
            isTopConnectShow: false,
            isBottomConnectShow: false
            }
        ],
        // 连线
        connectors: [
            {
            id: "path1",
            type: "Line",
            strokeW: 1,
            color: "#666",
            targetNode: {
                x: 50,
                y: 150,
                id: 2,
                width: 50,
                height: 50
            },
            sourceNode: {
                x: 300,
                y: 20,
                id: 1,
                width: 50,
                height: 50
            },
            isSelect: false,
            showDelete: false
            },
            {
            id: "path2",
            type: "Line",
            strokeW: 1,
            color: "#666",
            targetNode: {
                x: 200,
                y: 150,
                id: 3,
                width: 50,
                height: 50
            },
            sourceNode: {
                x: 300,
                y: 20,
                id: 1,
                width: 50,
                height: 50
            },
            isSelect: false,
            showDelete: false
            },
            {
            id: "path3",
            type: "Line",
            strokeW: 1,
            color: "#666",
            targetNode: {
                x: 400,
                y: 150,
                id: 4,
                width: 50,
                height: 50
            },
            sourceNode: {
                x: 300,
                y: 20,
                id: 1,
                width: 50,
                height: 50
            },
            isSelect: false,
            showDelete: false
            },
            {
            id: "path4",
            type: "Line",
            strokeW: 1,
            color: "#666",
            targetNode: {
                x: 200,
                y: 300,
                id: 5,
                width: 50,
                height: 50
            },
            sourceNode: {
                x: 400,
                y: 150,
                id: 4,
                width: 50,
                height: 50
            },
            isSelect: false,
            showDelete: false
            }
        ],
        gridData: [
            {
                x1: 0,
                x2: 100,
                y1: 20,
                y2: 20,
                color: "#c0c0c0",
                strokeWidth: 1,
                opacity: 0.3,
                id: 1
            },
            {
                x1: 0,
                x2: 100,
                y1: 40,
                y2: 40,
                color: "#c0c0c0",
                strokeWidth: 1,
                opacity: 0.3,
                id: 2
            },
            {
                x1: 0,
                x2: 100,
                y1: 60,
                y2: 60,
                color: "#c0c0c0",
                strokeWidth: 1,
                opacity: 0.3,
                id: 3
            },
            {
                x1: 0,
                x2: 100,
                y1: 80,
                y2: 80,
                color: "#c0c0c0",
                strokeWidth: 1,
                opacity: 0.3,
                id: 4
            },
            {
                x1: 20,
                x2: 20,
                y1: 0,
                y2: 100,
                color: "#c0c0c0",
                strokeWidth: 1,
                opacity: 0.3,
                id: 5
            },
            {
                x1: 40,
                x2: 40,
                y1: 0,
                y2: 100,
                color: "#c0c0c0",
                strokeWidth: 1,
                opacity: 0.3,
                id: 6
            },
            {
                x1: 60,
                x2: 60,
                y1: 0,
                y2: 100,
                color: "#c0c0c0",
                strokeWidth: 1,
                opacity: 0.3,
                id: 7
            },
            {
                x1: 80,
                x2: 80,
                y1: 0,
                y2: 100,
                color: "#c0c0c0",
                strokeWidth: 1,
                opacity: 0.3,
                id: 8
            },
            {
                x1: 100,
                x2: 100,
                y1: 0,
                y2: 100,
                color: "#c0c0c0",
                strokeWidth: 2,
                opacity: 0.6,
                id: 9
            },
            {
                x1: 0,
                x2: 100,
                y1: 100,
                y2: 100,
                color: "#c0c0c0",
                strokeWidth: 2,
                opacity: 0.6,
                id: 10
            }
        ]
    };
  },
  methods: {
    //拖拽shapeBar中的node 左侧的图片库
    dragShapeNode(nodeData,key,event){
      let NODE = nodeData[key]
      let toolbarName =NODE.type
      let toolbarIcon = NODE.icon
      let topoEle = $(`#topoId${this.topoId}`)
      console.log("topoEle",topoEle)
      let svgOffsetLeft = topoEle.find(".topoSvg").offset().left
      let svgOffsetTop =  topoEle.find(".topoSvg").offset().top
      let svgWidth =  topoEle.find(".topoSvg").width()
      let svgHeight =  topoEle.find(".topoSvg").height()
      let isContainSvgArea = false
      document.onmousemove = (event) =>{        
        let mouseX = event.clientX    //当前鼠标位置
        let mouseY = event.clientY 
        let nodeX = event.clientX - svgOffsetLeft + $(document).scrollLeft()+ this.svgAttr.viewX   //svg最终位置
        let nodeY = event.clientY - svgOffsetTop  + $(document).scrollTop() + this.svgAttr.viewY
        isContainSvgArea = false
        this.shapebarMoveNode.left = mouseX + 4 + $(document).scrollLeft()  // 鼠标位置 + 文档滚动的距离
        this.shapebarMoveNode.top =  mouseY + 4 + $(document).scrollTop()
        this.shapebarMoveNode.name = toolbarName
        this.shapebarMoveNode.icon = toolbarIcon
        this.shapebarMoveNode.isShow = true
        this.marker.isMarkerShow = false
        // 鼠标滑入svg区域内显示标尺并显示标尺正确位置
        if( mouseX >= svgOffsetLeft && 
            mouseX <= (svgOffsetLeft + svgWidth) && 
            mouseY >= (svgOffsetTop - $(document).scrollTop()) && 
            mouseY <= (svgOffsetTop+svgHeight - $(document).scrollTop())
          ){
          this.marker.isMarkerShow = true
          isContainSvgArea = true
          let n1 = Math.floor(nodeX / 20)  //grid宽高的整数倍
          let n2 = Math.floor(nodeY / 20)         
          this.marker.xmarkerY = n2 * 20
          this.marker.ymarkerX = n1 * 20
        }
      }
      document.onmouseup = (event)=>{
        document.onmousemove = null
        document.onmouseup = null
         // 判断鼠标在svg区域
        if(isContainSvgArea){
            let TOPODATA = this.topoData
            let type = NODE.type
            let name = NODE.type+'_'+ NODE.num
            NODE.num ++ 
            let id = GenNonDuplicateID(5)
            let nodeEndX = this.marker.ymarkerX
            let nodeEndY = this.marker.xmarkerY
            let svgNode ={
               name,
               type,
               id:id,
               x:nodeEndX,
               y:nodeEndY,
               icon:NODE.icon,
               width:NODE.width,
               height:NODE.height,
               initW:NODE.width,
               initH:NODE.height,           
               classType:NODE.classType,
               isLeftConnectShow:false,
               isRightConnectShow:false,
               containNodes:[],
               attrs:[]
            }
            this.marker.isMarkerShow = false    //标尺取消显示
            this.nodes.push(svgNode)   //创建一个svg Node    
            //计算是否与某个节点重叠
            for(let i =( this.nodes.length - 1); i >= 0; i-- ){
              let node = this.nodes[i]
              if(node.x <= nodeEndX && nodeEndX <= (node.x + node.width) && nodeEndY >= node.y && node.y + node.height >= nodeEndY && node.id != id){
                let canBeContain =  this.canConnectorTo(NODE.type,node.type,'Contain')  //判断是否能被包含在目标元素中
                if(canBeContain){
                  let connectorId = this.GenNonDuplicateID(3)
                  let connector={
                    id:connectorId,
                    type:'Contain',
                    sourceNode:{
                      id:id,
                    },
                    targetNode:{
                      id:node.id,
                    },
                    isSelect:false
                  }                                                 
                  TOPODATA.connectors.push(connector)            
                  node.containNodes.push(id)   //如果有嵌套关系，就在父节点放入子节点id        
                  this.refreshRowAndOuterNode(svgNode)  //刷新并列节点位置和父节点宽高 
                  this.refreshConnectorsData()  
                  break
                }          
              }
            }
        }    
        //重新初始toolbarMoveNode的值
        this.shapebarMoveNode.left = 0
        this.shapebarMoveNode.top =  0
        this.shapebarMoveNode.name = ''
        this.shapebarMoveNode.icon = ''
        this.shapebarMoveNode.isShow = false
      }
      //生成唯一id值
      function GenNonDuplicateID(randomLength){
        return Number(Math.random().toString().substr(3,randomLength) + Date.now()).toString(36)
      }
    },
    //svg工具栏选择工具
    selectToolbar(key){
      this.svgToolbar.forEach((ele,key) => {
        ele.isActive = false
      })
      this.svgToolbar[key].isActive = true
    },
    //编辑Topo图位置
    editTopo() {
      this.isEdit = true; // 编辑开关打开
    },
    //保存Topo图的数据
    saveTopoJson() {
      let data = {};
      let _nodes = JSON.parse(JSON.stringify(this.nodes));
      let _connectors = JSON.parse(JSON.stringify(this.connectors));
      data.nodes = _nodes;
      data.connectors = _connectors;
      // console.log("data:",data);
      var Save_button = document.getElementById("save_json");
      Save_button.addEventListener("click", saveHandler, false);
      function saveHandler() {
        var content = data;
        var content = JSON.stringify(data);
        // console.log("content",content);
        var blob = new Blob([content], { type: "text/plain;charset=utf-8" });
        saveAs(blob, "saveTopo.json");
      }
      this.isEdit = false; // 编辑关闭
    },
    //保存为png图片
    saveAsPng() {
      let node = document.querySelector("svg");
      let name = "svg";
      let width = this.svgAttr.width;
      let height = this.svgAttr.height;
      let type = "png";
      this.covertSVG2Image(node, name, width, height, (type = "png"));
    },
    /**
     * 将svg导出成图片
     * @param node svg节点 => document.querySelector('svg')
     * @param name 生成的图片名称
     * @param width 生成的图片宽度
     * @param height 生成的图片高度
     * @param type 生成的图片类型
     */
    covertSVG2Image(node, name, width, height, type = "png") {
      let serializer = new XMLSerializer();
      let source ='<?xml version="1.0" standalone="no"?>\r\n' + serializer.serializeToString(node);
      let image = new Image();
      image.src = "data:image/svg+xml;charset=utf-8," + encodeURIComponent(source);
      let canvas = document.createElement("canvas");
      canvas.width = width;
      canvas.height = height;
      let context = canvas.getContext("2d");
      context.fillStyle = "#fff";
      context.fillRect(0, 0, 10000, 10000);
      image.onload = function() {
        context.drawImage(image, 0, 0);
        let a = document.createElement("a");
        a.download = `${name}.${type}`;
        a.href = canvas.toDataURL(`image/${type}`);
        a.click();
      };
    },
    //打开本地static下的json文件，加载预览
    open_changeFile() {
      // //用axios的方法引入地址
      // this.$axios.get("http://tcc:9000/static/saveTopo.json").then(res => {
      //   console.log(res);
      //   if (res.status == "200") {
      //     // 赋值显示在页面
      //     this.fileData = JSON.parse(JSON.stringify(res.data));
      //     console.log("this.fileData:", this.fileData);
      //     this.connectors = this.fileData.connectors;
      //     this.nodes = this.fileData.nodes;
      //     this.$Message.success("本地json数据加载成功！");
      //   }
      // });
    },
    //鼠标滑过node
    mouseoverNode(key, event) {
      if (!this.isEdit) return;
      this.nodes[key].isTopConnectShow = true;
      this.nodes[key].isBottomConnectShow = true;
      // this.marker.xmarkerY = this.nodes[key].y
      // this.marker.ymarkerX = this.nodes[key].x
      this.getConnectLine(key);
    },
    //获取连线终点时的node的ID值
    getConnectLine(key) {
      if (!this.isEdit) return;
      this.connectingLine.endNode = this.nodes[key].id;
    },
    //鼠标划出顶部箭头时，将connectingLine.endNode再次初始化
    mouseoutLeftConnector(key) {
      if (!this.isEdit) return;
      this.connectingLine.endNode = "";
    },
    mouseoutNode(key, event) {
      if (!this.isEdit) return;
      this.nodes[key].isTopConnectShow = false;
      this.nodes[key].isBottomConnectShow = false;
    },
    // 手动画线
    drawConnectLine(key, event) {
      if (!this.isEdit) return;
      // 鼠标按下
      let CONNECTLINE = this.connectingLine; //绘制连线对象
      let CURNODE = this.nodes[key]; //当前点击node
      let nodeW = CURNODE.width; //当前node宽高
      let nodeH = CURNODE.height;
      let sourceNodeX = CURNODE.x;
      let sourceNodeY = CURNODE.y;
      let mouseX0 = event.clientX;
      let mouseY0 = event.clientY;
      let topoEle = $(`#topoId${this.topoId}`);
      let x1 = event.clientX - topoEle.find(".topoSvg").offset().left -2 +$(document).scrollLeft() +this.svgAttr.viewX; //连线开始位置的位置：鼠标点击的实际位置   为鼠标位置 - 当前元素的偏移值
      let y1 = event.clientY -topoEle.find(".topoSvg").offset().top +4 +$(document).scrollTop() +this.svgAttr.viewY;
      CONNECTLINE.isConnecting = true; //显示绘制连线
      CONNECTLINE.x1 = x1;
      CONNECTLINE.y1 = y1;
      CONNECTLINE.x2 = x1; //连线终点同样赋值为起点值
      CONNECTLINE.y2 = y1;
      CONNECTLINE.sourceNode = CURNODE.id; //将当前点击nodeid值赋给连线起点

      document.onmousemove = event => {
        let disX = event.clientX - mouseX0;
        let disY = event.clientY - mouseY0;
        let x2 = x1 + disX;
        let y2 = y1 + disY;
        CURNODE.isBottomConnectShow = true;
        CONNECTLINE.x2 = x2;
        CONNECTLINE.y2 = y2;
      };
      document.onmouseup = () => {
        document.onmousemove = null;
        document.onmouseup = null;
        let hasConnected = false; //标记是否已经有过连线
        let CONNECTORS = this.connectors;
        let sourceNodeW = nodeW;
        let sourceNodeH = nodeH;
        let targetNodeW = 0; //目标节点相关信息
        let targetNodeH = 0;
        let targetNodeX = 0;
        let targetNodeY = 0;
        let targetNodeType = "";
        let connectType = "";
        // console.log(CONNECTLINE.endNode);
        if (
          CONNECTLINE.endNode &&
          CONNECTLINE.endNode != CONNECTLINE.sourceNode
        ) {
          //正确连线：添加连线信息在connectors中
          //判断是否有已经有连线的情况
          CONNECTORS.forEach((item, index) => {
            if (
              item.sourceNode.id == CURNODE.id &&
              item.targetNode.id == CONNECTLINE.endNode &&
              item.type == "Line"
            )
              hasConnected = true;
          });
          //未连线情况下增加两者连线
          if (!hasConnected) {
            connectType = "Line";
            //获取目标节点宽高
            this.nodes.forEach((item, index) => {
              if (item.id == CONNECTLINE.endNode) {
                targetNodeW = item.width;
                targetNodeH = item.height;
                targetNodeX = item.x;
                targetNodeY = item.y;
                targetNodeType = item.type;
              }
            });
            let canLinkToTargetNode = this.canConnectorTo(
              CURNODE.type,
              targetNodeType,
              "Link"
            );
            if (!canLinkToTargetNode) {
              this.$message({
                showClose: true,
                message: CURNODE.type + "类型 不能连接 " + targetNodeType + "类型",
                type: "error"
              });
              CURNODE.isBottomConnectShow = false; //连线失败：起点右侧箭头暂且设置为消失
              CONNECTORS.forEach((item, key) => {
                //连线判断，如果已经有连线起点为当前的node，将起点箭头设置为显示
                this.nodes.forEach((node, key) => {
                  if (node.id == item.sourceNode.id && item.type == "Line")
                    node.isBottomConnectShow = true;
                });
              });
            } else {
              //类型：包含
              let connectorId = this.GenNonDuplicateID(3); // 生产唯一id
              let connector = {
                id: connectorId,
                type: connectType,
                strokeW: 1, //仅用于Line类型,默认3
                color: "#666", //仅用于Line类型，默认颜色
                targetNode: {
                  x: targetNodeX,
                  y: targetNodeY,
                  id: CONNECTLINE.endNode,
                  width: targetNodeW,
                  height: targetNodeH
                },
                sourceNode: {
                  x: sourceNodeX,
                  y: sourceNodeY,
                  id: CURNODE.id,
                  width: sourceNodeW,
                  height: sourceNodeH
                },
                showDelete: false,
                isSelect: false
              };
              CURNODE.isBottomConnectShow = true;
              this.nodes.forEach((item, key) => {
                if (item.id == CONNECTLINE.endNode)
                  item.isLeftConnectShow = true;
              });
              CONNECTORS.push(connector);
            }
          }
        } else {
          CURNODE.isBottomConnectShow = false; //连线失败：起点右侧箭头暂且设置为消失
          CONNECTORS.forEach((item, key) => {
            //连线判断，如果已经有连线起点为当前的node，将起点箭头设置为显示
            this.nodes.forEach((node, key) => {
              if (node.id == item.sourceNode.id && item.type == "Line")
                node.isBottomConnectShow = true;
            });
          });
        }
        //绘制连线恢复初始值
        CONNECTLINE.x1 = 0;
        CONNECTLINE.y1 = 0;
        CONNECTLINE.x2 = 0;
        CONNECTLINE.y2 = 0;
        CONNECTLINE.isConnecting = false;
        CONNECTLINE.sourceNode = "";
        CONNECTLINE.endNode = "";
      };
    },
    GenNonDuplicateID(randomLength) {
      return Number(
        Math.random()
          .toString()
          .substr(3, randomLength) + Date.now()
      ).toString(36);
    },
    canConnectorTo(curNodeType, connectorToNodeType, connectorType) {
      //当需要包含和连线规则的时候 清除以下注释
      // let canConnector = false
      // if(connectorType == 'Link'){
      //   this.connectorRules.forEach((ele,key)=>{
      //     if(ele.type == curNodeType){
      //       ele.canLinkToType.forEach((el,index)=>{
      //         if(el == connectorToNodeType) canConnector = true
      //       })
      //     }
      //   })
      // }else if(connectorType == 'Contain'){
      //   this.connectorRules.forEach((ele,key)=>{
      //     if(ele.type == curNodeType){
      //       ele.canBeContainedType.forEach((el,index)=>{
      //         if(el == connectorToNodeType) canConnector = true
      //       })
      //     }
      //   })
      // }
      let canConnector = true;
      return canConnector;
    },
    //拖拽svg中的node
    dragSvgNode(key, event) {
      if (!this.isEdit) return;
      let mouseX0 = event.clientX + $(document).scrollLeft(); //鼠标点击下的位置
      let mouseY0 = event.clientY + $(document).scrollTop();
      let CURNODE = this.nodes[key]; //点击的node对象
      let startX = CURNODE.x; //节点开始位置
      let startY = CURNODE.y;
      let curNodeId = CURNODE.id; //当前结点id
      let nodeW = CURNODE.width; //节点 宽高
      let nodeH = CURNODE.height;
      let nodeStartPosArr = [];
      // console.log('123',nodeStartPosArr); //拖拽时候的，每个图片的位置
      nodeStartPosArr.push({ id: CURNODE.id, x: CURNODE.x, y: CURNODE.y });
      this.nodes.forEach((node, key) => {
        // 关联属性设置框
        if (node.id == CURNODE.id) {
          this.selectNodeData = node;
        }
      });
      let endX, endY;
      document.onmousemove = event => {
        let disX = event.clientX - mouseX0 + $(document).scrollLeft(); //移动位置
        let disY = event.clientY - mouseY0 + $(document).scrollTop();
        endX = startX + disX; //最终位置
        endY = startY + disY;
        this.moveContianNode(disX, disY, nodeStartPosArr); //根据保存的数组数据移动相关节点
        this.refreshConnectorsData(); //及时更新连线数据
      };
      document.onmouseup = event => {
        document.onmousemove = null;
        document.onmouseup = null;
        this.marker.isMarkerShow = false; //隐藏标尺
        let NodeEndX = this.marker.ymarkerX; //最终位置为标尺的位置 最终节点位置
        let NodeEndY = this.marker.xmarkerY;
        //  let disX = NodeEndX - startX
        //  let disY = NodeEndY - startY
        let disX = endX - startX;
        let disY = endY - startY;
        let mouseDisX = event.clientX - mouseX0;
        let mouseDisY = event.clientY - mouseY0;
        // this.moveContianNode(disX,disY,nodeStartPosArr)  //移动包含着的子节点 不用
        //  this.drawContainLayout(CURNODE,NodeEndX,NodeEndY,true,nodeStartPosArr,mouseDisX,mouseDisY,startY)
        this.refreshConnectorsData(); //最后刷新连线
      };
    },
    //存入node及其子节点位置信息
    storeCurnodeStartPosition(CURNODE, startNodePosition) {
      startNodePosition.push({ id: CURNODE.id, x: CURNODE.x, y: CURNODE.y });
    },
    //contain情况下移动子节点位置
    moveContianNode(disX, disY, nodeStartPosArr) {
      if (!this.isEdit) return;
      // console.log(nodeStartPosArr)
      nodeStartPosArr.forEach((ele, key) => {
        let storeInfoId = ele.id;
        this.nodes.forEach((node, key) => {
          if (node.id == storeInfoId) {
            // console.log(storeInfoId, node.name)
            node.x = ele.x + disX;
            node.y = ele.y + disY;
          }
        });
      });
    },
    //刷新连线数据
    refreshConnectorsData() {
      this.connectors.forEach((item, index) => {
        //更新connectors里的数据
        this.nodes.forEach((node, key) => {
          if (item.sourceNode.id == node.id) {
            item.sourceNode.width = node.width;
            item.sourceNode.height = node.height;
            item.sourceNode.x = node.x;
            item.sourceNode.y = node.y;
          }
          if (item.targetNode.id == node.id) {
            item.targetNode.width = node.width;
            item.targetNode.height = node.height;
            item.targetNode.x = node.x;
            item.targetNode.y = node.y;
          }
        });
      });
    },
    deleteNode(key, event) {
      if (!this.isEdit) return;
      let node = this.nodes[key];
      this.deleteSelectNodeLink(node.id); // 删除与之连线
      this.nodes.splice(key, 1);
    },
    //删除node节点及其关系
    deleteNodeAndConnetor() {
      if (!this.isEdit) return;
      document.onkeydown = event => {
        let keycode = event.which; //键盘值
        if (keycode == 46 || keycode == 8) {
          //在mac上del的keycode是8,这样又会引起win下输入backspace也会删除
          //单节点和多选删除节点
          for (let i = 0; i < this.nodes.length; i++) {
            let node = this.nodes[i];
            if (node.isSelect) {
              this.deleteSelectNodeLink(node.id); // 删除 与node节点的连线
              let targetNodeId = "";
              let targetNode = null;
              // this.connectors.forEach((ele,key) => {
              //   if(ele.sourceNode.id == node.id ) targetNodeId = ele.targetNode.id
              // })
              // this.deleteCurNodeContainConnector(node)
              // if(targetNodeId){
              //   this.nodes.forEach((node,index) => {
              //   })
              // }
              this.nodes.splice(i, 1); // 删除节点
              //删除包含关系1.如果有父元素，恢复父元素的宽高位置
              // this.deleteCurnodeAndChildnodes(node) // 删除此节点内部所有包含的节点及其关系
              // this.deleteCurNodeContainConnector(node)   // 删除 node节点
              // this.refreshNodeArrows() //刷新节点的左右箭头展示
              i--;
              if (this.nodes.length > 0) {
                this.selectNodeIndex = null;
                this.selectNodeData = {};
              } else {
                this.selectNodeIndex = null;
                this.selectNodeData = {};
                this.isTopoAttrShow = false;
              }
            }
          }
        }
      };
    },
    //删除选中node的连线
    deleteSelectNodeLink(selectId) {
      if (!this.isEdit) return;
      let connectorObjArr = this.connectors;
      let connectorsLen = connectorObjArr.length;
      for (let i = 0; i < connectorsLen; i++) {
        let connectorObj = connectorObjArr[i];
        //删除连线
        if ( connectorObj.type == "Line" &&(connectorObj.sourceNode.id == selectId ||connectorObj.targetNode.id == selectId)) {
          this.connectors.splice(i, 1);
          i--;
          connectorsLen--;
        }
      }
    },
    // 单纯删除线
    deleteLine(key, event) {
      if (!this.isEdit) return;
      this.connectors.splice(key, 1);
      console.log(key);
    },
    //动态添加图片
    addNode(data) {
      if (!this.isEdit) return;
      // console.log(data)
      if (!data) return false;
      let node = {
        name: data.label,
        id: this.GenNonDuplicateID(6),
        x: 10,
        y: 10,
        icon: data.value,
        width: 50,
        height: 50,
        containNodes: [],
        attrs: [],
        classType: "T2",
        isTopConnectShow: false,
        isBottomConnectShow: false
      };

      this.$set(this.nodes, this.nodes.length, node);
      this.refreshConnectorsData();
      // console.log(this.nodes)
    },
    // 鼠标移入选中连线
    selectConnectorLine(key) {
      if (!this.isEdit) return;
      this.connectors[key].showDelete = true;
      // if(!this.editable) return false //如果非编辑状态 不可点击
      let connectors = this.connectors;
      let nodes = this.nodes;
      let selectLine = this.connectors[key];
      let lastIndex = connectors.length - 1;
      connectors.splice(key, 1);
      connectors.push(selectLine);
      //取消所有选中样式
      this.cancelAllNodesSelect();
      this.cancelAllLinksSelect();
      selectLine.isSelect = true;
      this.$set(connectors, lastIndex, selectLine);
      this.selectNodeData = selectLine; //将点击的连线信息赋值给属性面板
    },
    // 鼠标移出隐藏删除按钮
    cancelSelectConnectorLine(key) {
      this.connectors[key].showDelete = false;
    },
    //取消所有节点选中
    cancelAllNodesSelect() {
      this.nodes.forEach((ele, key) => {
        ele.isSelect = false;
        this.$set(this.nodes, key, ele);
      });
      this.selectNodeData = {};
    },
    //取消所有连线选中
    cancelAllLinksSelect() {
      this.connectors.forEach((ele, key) => {
        ele.isSelect = false;
        this.$set(this.connectors, key, ele);
      });
      this.selectNodeData = {};
    },

    //1.取消选中的node节点 2. 移动viewbox
    mousedownTopoSvg(event) {
      if (!this.isEdit) return;
      // console.log(this.svgAttr);
      let mouseX0 = event.clientX; //鼠标点击下的位置
      let mouseY0 = event.clientY;
      let startViewX = this.svgAttr.viewX;
      let startViewY = this.svgAttr.viewY;
      let startSvgW = this.svgAttr.width;
      let startSvgH = this.svgAttr.height;
      let svgMinW = this.svgAttr.minW;
      let svgMinH = this.svgAttr.minH;
      let selectionBoxX = 0;
      let selectionBoxY = 0;
      this.cancelAllNodesSelect(); //取消所有节点选中
      this.cancelAllLinksSelect(); //取消连线选中

      //移动viewbox位置
      document.onmousemove = event => {
        let disX = event.clientX - mouseX0;
        let disY = event.clientY - mouseY0;
        let endSvgW = startSvgW - disX;
        let endSvgH = startSvgH - disY;
        this.svgAttr.isHand = true;
        this.svgAttr.viewX = startViewX <= disX ? 0 : startViewX - disX; //根据鼠标移动的位移，得到视图移动位移
        this.svgAttr.viewY = startViewY <= disY ? 0 : startViewY - disY;
        this.svgAttr.width = endSvgW < svgMinW ? svgMinW : endSvgW; // 动态设置svg宽高
        this.svgAttr.height = endSvgH < svgMinH ? svgMinH : endSvgH;
        this.marker.xmarkerX = this.svgAttr.width;
        this.marker.ymarkerY = this.svgAttr.height;
      };
      document.onmouseup = event => {
        document.onmousemove = null;
        document.onmouseup = null;
        this.svgAttr.isHand = false;
        this.svgAttr.isCrosshair = false;
      };
    },
    //画布(放大/缩小)方法
    mouseWheel(event) {
      // console.log(event)
      let i = 0.1;
      if (this.rate < 0) return;

      if (event.deltaY > 0) {
        // 缩小
        this.rate = this.rate - 0.1;
        if (this.rate <= 0.6) return;
      } else {
        // 放大
        this.rate = this.rate + 0.1;
        if (this.rate >= 1.8) return;
      }

      this.scale = "scale(" + this.rate + ")";
      console.log(this.scale, event.deltaY);
    }
  },
  created() {},
  destroyed() {
    // 销毁 panZoomTiger 实例
  },
  computed: {
    //   scale: function(){
    //     return this.mouseWheel()
    //   }
  },
  mounted() {
    this.deleteNodeAndConnetor();
  }
};
</script>

<style scoped>
    /*移动的小图片样式：node*/
    .shapeIcon{ text-align: center;-webkit-user-select:none;user-select:none;}
    .shapeIconImg{ width: 28px;height: 28px;-webkit-user-select:none;user-select:none;}
    .shapeName{font-size:12px;text-align: center;padding:0 5px;text-overflow:ellipsis;overflow:hidden;white-space:nowrap;-webkit-user-select:none;user-select:none;color:#000}
    .moveNode{ position: absolute;border:1px solid #768699;box-sizing: border-box;}
    .nodeMoveCss{ width:57px;height: 57px;background-color: #fff;-webkit-user-select:none;user-select:none;box-sizing: border-box;padding:5px;}
    /* header部分 */
    .svgHeadItemLst{
        display: flex;
        height: 32px;
    }
    .svgHeadItemLst .svgHeadItem {
        padding: 8px 10px;
        border: 1px solid #aaaaaa;
        border-radius: 4px;
        cursor: pointer;
        list-style: none;
        /* border-left-width: 0; */
    }
    .svgHeadItemLst .svgHeadItem.active {
        /* background-color: #ebebeb; */
        -webkit-box-shadow: 2px 2px 1px #ccc inset;
        box-shadow: 2px 2px 1px #ccc inset;
    }
    .svgHeadItem:first-child {
        /* border-left-width: 1px; */
        margin-right: 10px;
    }
    .svgHeadItemLst .svgHeadItem .svgHeadItemImg.toolbar-default {
        background-position: -16px 0px;
    }
    .svgHeadItemLst .svgHeadItem .svgHeadItemImg {
        background: url(/static/img/icons.png);
        width: 16px;
        height: 16px;
        background-size: 479px 16px;
    }
    .svgHeadItemLst .svgHeadItem .svgHeadItemImg.toolbar-rectangle_selection {
        background-position: -294px 0px;
    }
    /*中间区域*/
    .svgMain {
        height: 100%;
        box-sizing: border-box;
        display: flex;
        flex: 1;
        border: 1px solid #e3e3e3;
        margin-top: 10px;
    }
    /* 中间区域右侧操作 */
    .topoWrap {
        width: 1410px;
        height: 700px;
        overflow: hidden;
        border-left: 1px solid #e3e3e3;
        background-color: #fff;
        /* border-radius: 6px;
        margin-top: 10px; */
    }
    .topoSvg {
        user-select: none;
    }
    .reactClass {
        stroke-width: 1;
        stroke: #666;
        stroke-dasharray: 2 3;
        fill: transparent;
        cursor: default;
    }
    .connectorsG.active {
        /* filter: url(#f1) */
    }
</style>