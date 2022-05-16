<template>
  <div id="flowChart"></div>
  <button @click="addNewNode">添加一个节点</button>
  <button @click="deleteNode">删除一个节点</button>
  <button @click="addNewLink">添加一条线</button>
  <button @click="deleteLink">删除一条线</button>
  <div>
    <span>节点名称</span>
    <input type="text" v-model="nodeName">
  </div>
  <div>
    <span>节点位置</span>
    <input type="text" v-model="nodeLoc">
  </div>
  <div>
    <span>节点父级</span>
    <input type="text" v-model="nodeParent">
  </div>
  <div>
    <span>节点子级</span>
    <input type="text" v-model="nodeChild">
  </div>
</template>

<script setup>
import go from "gojs";
import { onMounted, reactive, toRefs } from "vue";
let myDiagram = null;

const data = reactive({
  nodeName: "",
  nodeLoc: "",
  nodeParent: "",
  nodeChild: ""
})

const {
  nodeName,
  nodeLoc,
  nodeParent,
  nodeChild
} = toRefs(data)

const addNewNode = () => {
  const uuid = window.crypto.randomUUID();
  if (!data.nodeName) {
    alert("请输入节点名")
    return;
  }
  const newNode = { text: data.nodeName, key: uuid, loc: data.nodeLoc, category: "circle" };
  myDiagram.model.addNodeData(newNode);
  if (!data.nodeParent && !data.nodeChild) {
    return;
  }
  const allNode = JSON.parse(myDiagram.model.toJson());
  const nodeList = allNode.nodeDataArray;
  if (data.nodeParent && !data.nodeChild) {
    const key = nodeList.find(e => e.text.includes(data.nodeParent)).key;
    myDiagram.model.addLinkData({ from: key, to: uuid });
    return;
  }
  if (!data.nodeParent && data.nodeChild) {
    const key = nodeList.find(e => e.text.includes(data.nodeChild)).key;
    myDiagram.model.addLinkData({ from: uuid, to: key });
    return;
  }
  const parentKey = nodeList.find(e => e.text.includes(data.nodeParent)).key;
  const childKey = nodeList.find(e => e.text.includes(data.nodeChild)).key;
  myDiagram.model.addLinkData({ from: parentKey, to: uuid });
  myDiagram.model.addLinkData({ from: uuid, to: childKey });
}

const deleteNode = () => {
  myDiagram.commandHandler.deleteSelection();
}

const addNewLink = () => {
  const allNode = JSON.parse(myDiagram.model.toJson());
  const nodeList = allNode.nodeDataArray;
  const parentKey = nodeList.find(e => e.text.includes(data.nodeParent)).key;
  const childKey = nodeList.find(e => e.text.includes(data.nodeChild)).key;
  myDiagram.model.addLinkData({ from: parentKey, to: childKey });
}

const deleteLink = () => {
  myDiagram.commandHandler.deleteSelection();
}

onMounted(() => {
  myDiagram = initDiagram("flowChart");
  addNodeTemplate(myDiagram);
  add(myDiagram);
});

const initDiagram = (dom) => {
  if (!dom) return "dom信息无效";
  return new go.Diagram(dom, { "undoManager.isEnabled": true });
};

const addNodeTemplate = (myDiagram) => {
  myDiagram.nodeTemplateMap.add(
    "circle",
    new go.Node("Auto")
      .bind("location", "loc", go.Point.parse)
      .add(new go.Shape("Circle", { width: 150, height: 150, fill: "#79C900", stroke: null }).bind("figure", "fig"))
      .add(new go.TextBlock("默认文字", { font: "bold 20px Helvetica, bold Arial, sans-serif", stroke: "white" }).bind("text", "text"))
  );
  myDiagram.nodeTemplateMap.add(
    "square",
    new go.Node("Auto")
      .bind("location", "loc", go.Point.parse)
      .add(new go.Shape("RoundedRectangle", { width: 150, height: 150, fill: "#DC3C00", stroke: null }).bind("figure", "fig"))
      .add(new go.TextBlock("默认文字", { font: "bold 20px Helvetica, bold Arial, sans-serif", stroke: "white" }).bind("text", "text"))
  );
  myDiagram.linkTemplateMap.add(
    "adopt",
    new go.Link({ routing: go.Link.Orthogonal, corner: 5, selectionChanged: selectNode })
      .add(new go.Shape({ strokeDashArray: [10, 20] }).bind("stroke", "stroke").bind("strokeWidth", "strokeWidth")) // 线段模板
      .add(new go.Shape({ stroke: 'transparent', strokeWidth: 0 }).bind("toArrow", "toArrow").bind("fill", "arrowfill")) // 箭头模板
      .add(new go.Panel("Auto").bind("alignmentFocus", "textPos")
        .add(new go.Shape({ fill: 'transparent', cursor: "pointer" }).bind("stroke", "stroke"))
        .add(new go.TextBlock("默认文字", { margin: 10 }).bind("text", "label").bind("stroke", "stroke"))) // 文字块
  )
};

const add = (myDiagram) => {
  const nodeList = [
    { text: "提交申请", key: "1", loc: "0 0", category: "circle" },
    { text: "主管审批", key: "2", loc: "300 200", category: "square" },
    { text: "总监审批", key: "3", loc: "600 0", category: "square" },
  ]
  const linkList = [
    { category: 'adopt', from: "1", to: "2", stroke: "orange", strokeWidth: "2", toArrow: "Standard", arrowfill: "orange", label: "驳回", textPos: new go.Spot(0, 0, 0, -120) },
    { category: 'adopt', from: "2", to: "3", stroke: "green", strokeWidth: "2", toArrow: "Standard", arrowfill: "green", label: "通过", textPos: new go.Spot(1, 0, 0, -120) }
  ]
  myDiagram.model = new go.GraphLinksModel(nodeList, linkList);
}

const selectNode = (node) => {
  if (node.isSelected) {
    if (node.data.label === "驳回") {
      alert("驳回成功");
    } else if (node.data.label === "通过") {
      alert("成功通过");
    }
    node.isSelected = false;
  }
}
</script>

<style scoped>
#flowChart {
  width: 1200px;
  height: 600px;
  margin: 0 auto;
  background-color: #dae4e4;
}
</style>