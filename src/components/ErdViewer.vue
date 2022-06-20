<template>
  <div class="erd-viewer">
    <div id="diagram"></div>
  </div>
</template>

<script>
import go from 'gojs';

const DIV_NAME = 'diagram';
const $ = go.GraphObject.make;

function createDiagram() {
  return $(
    go.Diagram,
    DIV_NAME, // create a Diagram for the DIV HTML element
    {
      // enable undo & redo
      'undoManager.isEnabled': true,
    },
  );
}

function setModel(diagram, nodeData, linkData) {
  diagram.model = new go.GraphLinksModel(nodeData, linkData);
}

function setNodeTemplate(diagram) {
  diagram.nodeTemplate = $(
    go.Node,
    'Auto', // the Shape will go around the TextBlock
    $(
      go.Shape,
      'RoundedRectangle',
      { strokeWidth: 0, fill: 'white' }, // default fill is white
      // Shape.fill is bound to Node.data.color
      new go.Binding('fill', 'color'),
    ),
    $(
      go.TextBlock,
      { margin: 8 }, // some room around the text
      // TextBlock.text is bound to Node.data.key
      new go.Binding('text', 'key'),
    ),
  );
}

export default {
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const myDiagram = createDiagram();

      setNodeTemplate(myDiagram);

      const nodeData = [
        { key: 'Alpha', color: 'lightblue' },
        { key: 'Beta', color: 'orange' },
        { key: 'Gamma', color: 'lightgreen' },
        { key: 'Delta', color: 'pink' },
      ];

      const linkData = [
        { from: 'Alpha', to: 'Beta' },
        { from: 'Alpha', to: 'Gamma' },
        { from: 'Beta', to: 'Beta' },
        { from: 'Gamma', to: 'Delta' },
        { from: 'Delta', to: 'Alpha' },
      ];

      setModel(myDiagram, nodeData, linkData);
    },
  },
};
</script>

<style scoped>
#diagram {
  width: 100%;
  height: 500px;
}
</style>
