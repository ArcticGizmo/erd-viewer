<template>
  <div class="erd-viewer">
    <div class="actions">
      Actions
    </div>
    <div ref="container" class="container"></div>
  </div>
</template>

<script>
import go from 'gojs';

const DIV_NAME = 'diagram';
const $ = go.GraphObject.make;

function createDiagram() {
  return $(go.Diagram, DIV_NAME, {
    'undoManager.isEnabled': true,
    allowDelete: false,
    allowCopy: false,
    layout: $(go.ForceDirectedLayout),
    'undoManager.isEnabled': true,
  });
}

function setModel(diagram, nodeDataArray, linkDataArray) {
  diagram.model = new go.GraphLinksModel({
    copiesArrays: true,
    copiesArrayObjects: true,
    nodeDataArray,
    linkDataArray,
  });
}

function setNodeTemplate(diagram) {
  diagram.nodeTemplate = $(
    go.Node,
    'Auto', // the whole node panel
    {
      selectionAdorned: true,
      resizable: true,
      layoutConditions: go.Part.LayoutStandard & ~go.Part.LayoutNodeSized,
      fromSpot: go.Spot.AllSides,
      toSpot: go.Spot.AllSides,
      isShadowed: true,
      shadowOffset: new go.Point(3, 3),
      shadowColor: '#C5C1AA',
    },
    new go.Binding('location', 'location').makeTwoWay(),
    // whenever the PanelExpanderButton changes the visible property of the "LIST" panel,
    // clear out any desiredSize set by the ResizingTool.
    new go.Binding('desiredSize', 'visible', v => new go.Size(NaN, NaN)).ofObject('LIST'),
    // define the node's outer shape, which will surround the Table
    $(go.Shape, 'RoundedRectangle', { fill: 'white', stroke: '#eeeeee', strokeWidth: 3 }),
    $(
      go.Panel,
      'Table',
      { margin: 8, stretch: go.GraphObject.Fill },
      $(go.RowColumnDefinition, { row: 0, sizing: go.RowColumnDefinition.None }),
      // the table header
      $(
        go.TextBlock,
        {
          row: 0,
          alignment: go.Spot.Center,
          margin: new go.Margin(0, 24, 0, 2), // leave room for Button
          font: 'bold 16px sans-serif',
        },
        new go.Binding('text', 'key'),
      ),
      // the collapse/expand button
      $(
        'PanelExpanderButton',
        'LIST', // the name of the element whose visibility this button toggles
        { row: 0, alignment: go.Spot.TopRight },
      ),
      // the list of Panels, each showing an attribute
      $(
        go.Panel,
        'Vertical',
        {
          name: 'LIST',
          row: 1,
          padding: 3,
          alignment: go.Spot.TopLeft,
          defaultAlignment: go.Spot.Left,
          stretch: go.GraphObject.Horizontal,
          itemTemplate: buildItemTemplate(),
        },
        new go.Binding('itemArray', 'items'),
      ),
    ), // end Table Panel
  ); // end Node
}

function setLinkTemplate(diagram) {
  diagram.linkTemplate = $(
    go.Link, // the whole link panel
    {
      selectionAdorned: true,
      layerName: 'Foreground',
      reshapable: true,
      routing: go.Link.AvoidsNodes,
      corner: 5,
      curve: go.Link.JumpOver,
    },
    $(
      go.Shape, // the link shape
      { stroke: '#303B45', strokeWidth: 2.5 },
    ),
    $(
      go.TextBlock, // the "from" label
      {
        textAlign: 'center',
        font: 'bold 14px sans-serif',
        stroke: '#1967B3',
        segmentIndex: 0,
        segmentOffset: new go.Point(NaN, NaN),
        segmentOrientation: go.Link.OrientUpright,
      },
      new go.Binding('text', 'text'),
    ),
    $(
      go.TextBlock, // the "to" label
      {
        textAlign: 'center',
        font: 'bold 14px sans-serif',
        stroke: '#1967B3',
        segmentIndex: -1,
        segmentOffset: new go.Point(NaN, NaN),
        segmentOrientation: go.Link.OrientUpright,
      },
      new go.Binding('text', 'toText'),
    ),
  );
}

function buildItemTemplate() {
  return $(
    go.Panel,
    'Horizontal',
    $(
      go.Shape,
      {
        desiredSize: new go.Size(15, 15),
        strokeJoin: 'round',
        strokeWidth: 3,
        stroke: null,
        margin: 2,
      },
      new go.Binding('figure', 'figure'),
      new go.Binding('fill', 'color'),
      new go.Binding('stroke', 'color'),
    ),
    $(
      go.TextBlock,
      {
        stroke: '#333333',
        font: 'bold 14px sans-serif',
      },
      new go.Binding('text', 'name'),
    ),
  );
}

export default {
  props: {
    nodeData: { type: Array, default: () => [] },
    linkData: { type: Array, default: () => [] },
  },
  data() {
    return {
      savedState: null,
      diagram: null,
    };
  },
  watch: {
    nodeData: {
      immediate: false,
      handler() {
        this.update();
      },
    },
  },
  mounted() {
    this.update();
  },
  methods: {
    update() {
      const wrapper = this.$refs.container;
      while (wrapper.firstChild) {
        wrapper.removeChild(wrapper.firstChild);
      }

      const div = document.createElement('div', { id: 'diagram' });
      div.setAttribute('id', DIV_NAME);
      div.style.width = '100%';
      div.style.height = '100%';
      wrapper.appendChild(div);

      const diagram = createDiagram();

      setNodeTemplate(diagram);
      setLinkTemplate(diagram);

      setModel(diagram, this.nodeData, this.linkData);

      this.diagram = diagram;
    },
  },
};
</script>

<style scoped>
.erd-viewer .actions {
  display: flex;
}

.container {
  width: 100%;
  height: 600px;
  border: 0.1px solid gray;
}
</style>
