<template>
  <div id="app">
    <div ref="graphGrid"
      style="position:absolute;overflow:auto;top:36px;bottom:0px;left:0px;right:0px;border-top:gray 1px solid;">
    </div>
  </div>
</template>

<script>
// import mxgraph
const mxgraph = require("mxgraph")({
  mxImageBasePath: "@/node_modules/mxgraph/javascript/src/images",
  mxBasePath: "@/node_modules/mxgraph/javascript/src"
});

const { mxEvent, mxClient, mxGraph, mxRubberband, mxUtils, mxCodec, mxCellTracker,
        mxConstants, mxPerimeter, mxFastOrganicLayout } = mxgraph;

// Parses the mxGraph XML file format
function read(graph, filename)
{
  var req = mxUtils.load(filename);
  // console.log(req);
  var root = req.getDocumentElement();
  console.log(root.ownerDocument);
  var dec = new mxCodec(root.ownerDocument);
  console.log(dec);
  
  dec.decode(root, graph.getModel());
};

// 创建graph初始化函数
function initGraph(container)
{
  // Checks if the browser is supported
  if (!mxClient.isBrowserSupported())
  {
    mxUtils.error('Browser is not supported!', 200, false);
  }
  else
  {
    // Creates the graph inside the given container
    var graph = new mxGraph(container);
    
    graph.setEnabled(false);
    graph.setPanning(true);
    graph.setTooltips(true);
    graph.panningHandler.useLeftButtonForPanning = true;
    // Adds a highlight on the cell under the mousepointer
    new mxCellTracker(graph);
    
    // Changes the default vertex style in-place
    var style = graph.getStylesheet().getDefaultVertexStyle();
    style[mxConstants.STYLE_SHAPE] = mxConstants.SHAPE_ROUNDED;
    style[mxConstants.STYLE_PERIMETER] = mxPerimeter.RectanglePerimeter;
    style[mxConstants.STYLE_GRADIENTCOLOR] = 'white';
    style[mxConstants.STYLE_PERIMETER_SPACING] = 4;
    style[mxConstants.STYLE_SHADOW] = true;
    
    style = graph.getStylesheet().getDefaultEdgeStyle();
    style[mxConstants.STYLE_LABEL_BACKGROUNDCOLOR] = 'white';
            
    style = mxUtils.clone(style);
    style[mxConstants.STYLE_STARTARROW] = mxConstants.ARROW_CLASSIC;
    graph.getStylesheet().putCellStyle('2way', style);
    
    graph.isHtmlLabel = function(cell)
    {
      return true;
    };
				
    // Larger grid size yields cleaner layout result
    graph.gridSize = 20;
  
    // Creates a layout algorithm to be used
    // with the graph
    var layout = new mxFastOrganicLayout(graph);

    // Moves stuff wider apart than usual
    layout.forceConstant = 140;

    // Load cells and layouts the graph
    graph.getModel().beginUpdate();
    try
    {	
      // Loads the mxGraph file format (XML file)
      read(graph, '/stencils.xml');
              
      // Gets the default parent for inserting new cells. This
      // is normally the first child of the root (ie. layer 0).
      var parent = graph.getDefaultParent();

      // Executes the layout
      layout.execute(parent);
    }
    finally
    {
      // Updates the display
      graph.getModel().endUpdate();
    }
  }
};


export default {
  name: 'app',
  mounted () {
    initGraph(this.$refs.graphGrid)
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
