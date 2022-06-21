<template>
  <div class="erd-viewer">
    <div class="actions">
      <button @click="onSave">Save</button>
      <button @click="onLoad">Load</button>
    </div>
    <div ref="diagram" id="diagram"></div>
  </div>
</template>

<script>
import go from 'gojs';

const DIV_NAME = 'diagram';
const $ = go.GraphObject.make;
const COLORS = {
  red: '#be4b15',
  green: '#52ce60',
  blue: '#6ea5f8',
  lightred: '#fd8852',
  lightblue: '#afd4fe',
  lightgreen: '#b9e986',
  pink: '#faadc1',
  purple: '#d689ff',
  orange: '#fdb400',
};

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

function getData() {
  const nodeDataArray = [
    {
      key: 'Products',
      items: [
        { name: 'ProductID', iskey: true, figure: 'Decision', color: COLORS.red },
        { name: 'ProductName', iskey: false, figure: 'Hexagon', color: COLORS.blue },
        { name: 'SupplierID', iskey: false, figure: 'Decision', color: 'purple' },
        { name: 'CategoryID', iskey: false, figure: 'Decision', color: 'purple' },
      ],
    },
    {
      key: 'Suppliers',
      items: [
        { name: 'SupplierID', iskey: true, figure: 'Decision', color: COLORS.red },
        { name: 'CompanyName', iskey: false, figure: 'Hexagon', color: COLORS.blue },
        { name: 'ContactName', iskey: false, figure: 'Hexagon', color: COLORS.blue },
        { name: 'Address', iskey: false, figure: 'Hexagon', color: COLORS.blue },
      ],
    },
    {
      key: 'Categories',
      items: [
        { name: 'CategoryID', iskey: true, figure: 'Decision', color: COLORS.red },
        { name: 'CategoryName', iskey: false, figure: 'Hexagon', color: COLORS.blue },
        { name: 'Description', iskey: false, figure: 'Hexagon', color: COLORS.blue },
        { name: 'Picture', iskey: false, figure: 'TriangleUp', color: COLORS.pink },
      ],
    },
    {
      key: 'Order Details',
      items: [
        { name: 'OrderID', iskey: true, figure: 'Decision', color: COLORS.red },
        { name: 'ProductID', iskey: true, figure: 'Decision', color: COLORS.red },
        { name: 'UnitPrice', iskey: false, figure: 'Circle', color: COLORS.green },
        { name: 'Quantity', iskey: false, figure: 'Circle', color: COLORS.green },
        { name: 'Discount', iskey: false, figure: 'Circle', color: COLORS.green },
      ],
    },
  ];

  const linkDataArray = [
    { from: 'Products', to: 'Suppliers', text: '0..N', toText: '1' },
    { from: 'Products', to: 'Categories', text: '0..N', toText: '1' },
    { from: 'Order Details', to: 'Products', text: '0..N', toText: '1' },
  ];

  return { nodeDataArray, linkDataArray };
}

export default {
  props: {
    nodeData: { type: Array, default: () => [] },
    linkData: { type: Array, default: () => [] },
  },
  data() {
    return {
      savedState: null,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const myDiagram = createDiagram();

      setNodeTemplate(myDiagram);
      setLinkTemplate(myDiagram);

      setModel(myDiagram, this.nodeData, this.linkData);
    },
    onSave() {
      console.dir(this.$refs.diagram.value);
    },
    onLoad() {},
  },
};
</script>

<style scoped>
.erd-viewer .actions {
  display: flex;
}

#diagram {
  width: 100%;
  height: 600px;
}
</style>
