<template>
  <div>

    <div class="header">
      <button :disabled="disableRunBtn" class="run-btn btn" @click="runDijkstra()">Run Dijkstra</button>
      <button :disabled="disableResetBtn" class="reset-btn btn" @click="reset()">Reset</button>
    </div>

    <div id="pathfinder">
      <div class='grid' v-for="(row, rowIdx) in grid" v-bind:key="rowIdx">
        <div div v-for="(node, nodeIdx) in row" v-bind:key="nodeIdx"
             class="node"
             @mousedown="handleMouseDown(node.row, node.col)"
             @mouseenter="handleMouseEnter(node.row, node.col)"
             @mouseup="handleMouseUp"
             :class="{
              'is-start-node' : node.isStart,
              'is-finish-node' : node.isFinish,
              'is-visited-node' : node.isVisited,
              'is-path' : node.isPath,
              'is-wall' : node.isWall,
             }">
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Pathfinder',
    data() {
      return {
        mouseIsPressed: false,
        disableRunBtn: false,
        disableResetBtn: false,
        grid: [],
        rowCount: 20,
        colCount: 50,
        START_ROW: 10,
        START_COL: 15,
        FINISH_ROW: 10,
        FINISH_COL: 35
      }
    },
    beforeMount() {
      this.initGrid();
    },
    methods: {
      initGrid() {
        for (let row = 0; row < this.rowCount; row++) {
          this.grid[row] = [];
          for (let col = 0; col < this.colCount; col++) {
            this.grid[row][col] = this.createNode(col, row);
          }
        }
      },
      reset() {
        this.initGrid();
        this.grid = [...this.grid];
        this.disableRunBtn = false;
      },
      animateDijkstra(visitedNodesInOrder, shortestPathNodesInOrder) {
        for (let i = 0; i < visitedNodesInOrder.length; i++) {
          setTimeout(() => {
            const node = visitedNodesInOrder[i];
            this.grid[node.row][node.col].isVisited = true;
            this.grid = [...this.grid];
            if (visitedNodesInOrder.length - 1 === i) {
              for (let j = 0; j < shortestPathNodesInOrder.length; j++) {
                setTimeout(() => {
                  const innerNode = shortestPathNodesInOrder[j];
                  this.grid[innerNode.row][innerNode.col].isPath = true;
                  this.grid = [...this.grid];
                  if (shortestPathNodesInOrder.length - 1 === j) {
                    this.disableResetBtn = false;
                  }
                }, 30 * j)
              }
            }
          }, 5 * i)
        }
      },
      handleMouseDown(row, col) {
        this.mouseIsPressed = true;
        this.getNewGridWithWallToggled(row, col);
      },
      handleMouseEnter(row, col) {
        if (this.mouseIsPressed) {
          this.getNewGridWithWallToggled(row, col);
        }
      },
      handleMouseUp() {
        this.mouseIsPressed = false;
      },
      getNewGridWithWallToggled(row, col) {
        this.grid[row][col].isWall = true;
        this.grid = [...this.grid];
      },
        runDijkstra() {
        this.disableRunBtn = true;
        this.disableResetBtn = true;

        const startNode = this.grid[this.START_ROW][this.START_COL];
        const finishNode = this.grid[this.FINISH_ROW][this.FINISH_COL];

        let visitedNodesInOrder = this.dijkstra(this.grid, startNode, finishNode);
        let shortestPathNodesInOrder = this.getNodesInShortestPathOrder(finishNode);

        for (let i = 0; i < this.grid.length; i++) {
          for (let j = 0; j < this.grid[i].length; j++) {
            this.grid[i][j].isVisited = false;
          }
        }
        this.animateDijkstra(visitedNodesInOrder, shortestPathNodesInOrder);
      },
    createNode(col, row){
   return {
    row,
    col,
    isStart: col === this.START_COL && row === this.START_ROW,
    isFinish: col === this.FINISH_COL && row === this.FINISH_ROW,
    distance: Infinity,
    isPath: false,
    isVisited: false,
    isWall: false,
    previousNode: null
  }
 },
//###########################//DIJKSTRA//##################################//

dijkstra(grid, startNode, finishNode) {
  const visitedNodesInOrder = [];
  startNode.distance = 0;
  const unvisitedNodes = this.getAllNodes(grid);
  while (unvisitedNodes.length) {
    this.sortNodesByDistance(unvisitedNodes);
    const closestNode = unvisitedNodes.shift();
    // If we encounter a wall, we skip it.
    if (closestNode.isWall) continue;
    // If the closest node is at a distance of infinity,
    // we must be trapped and should therefore stop.
    if (closestNode.distance === Infinity) return visitedNodesInOrder;
    closestNode.isVisited = true;
    visitedNodesInOrder.push(closestNode);
    if (closestNode === finishNode) return visitedNodesInOrder;
    this.updateUnvisitedNeighbors(closestNode, grid);
  }
},
sortNodesByDistance(unvisitedNodes) {
  unvisitedNodes.sort((nodeA, nodeB) => nodeA.distance - nodeB.distance);
},
updateUnvisitedNeighbors(node, grid) {
  const unvisitedNeighbors = this.getUnvisitedNeighbors(node, grid);
  for (const neighbor of unvisitedNeighbors) {
    neighbor.distance = node.distance + 1;
    neighbor.previousNode = node;
  }
},
getUnvisitedNeighbors(node, grid) {
  const neighbors = [];
  const {col, row} = node;
  if (row > 0) neighbors.push(grid[row - 1][col]);
  if (row < grid.length - 1) neighbors.push(grid[row + 1][col]);
  if (col > 0) neighbors.push(grid[row][col - 1]);
  if (col < grid[0].length - 1) neighbors.push(grid[row][col + 1]);
  return neighbors.filter(neighbor => !neighbor.isVisited);
},
getAllNodes(grid) {
  const nodes = [];
  for (const row of grid) {
    for (const node of row) {
      nodes.push(node);
    }
  }
  return nodes;
},
getNodesInShortestPathOrder(finishNode) {
  const nodesInShortestPathOrder = [];
  let currentNode = finishNode;
  while (currentNode !== null) {
    nodesInShortestPathOrder.unshift(currentNode);
    currentNode = currentNode.previousNode;
  }
  return nodesInShortestPathOrder;
}
    }
  }
</script>


<style scoped>
  #pathfinder {
    font-size: 0;
    border: none;
    width: auto;
    margin: 0 auto;
  }
.node {
      width: 35.8px;
      height: 35.58px;
      background-color:gainsboro;
      border: 1px solid black;
      display: inline-block;
      text-align: center;
}

.is-finish-node{
  background-color: red;
}

.is-start-node {
  background-color: green;
}
  .node {
  width: 25px;
  height: 25px;
  outline: 1px solid rgb(175, 216, 248);
  display: inline-block;
}


.is-visited-node{
  animation-name: visitedAnimation;
  animation-duration: 1.5s;
  animation-timing-function: ease-out;
  animation-delay: 0;
  animation-direction: alternate;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  animation-play-state: running;
}

@keyframes visitedAnimation {
  0% {
    transform: scale(0.3);
    background-color: rgba(0, 0, 66, 0.75);
    border-radius: 100%;
  }

  50% {
    background-color: rgba(17, 104, 217, 0.75);
  }

  75% {
    transform: scale(1.2);
    background-color: rgba(0, 217, 159, 0.75);
  }

  100% {
    transform: scale(1);
    background-color: rgba(0, 190, 218, 0.75);
  }
}

.is-wall{
  background-color: rgb(12, 53, 71);
}

.is-path{
  animation-name: shortestPath;
  animation-duration: 1.5s;
  animation-timing-function: ease-out;
  animation-delay: 0;
  animation-direction: alternate;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  animation-play-state: running;
}

@keyframes shortestPath {
  0% {
    transform: scale(0.6);
    background-color: rgb(255, 254, 106);
  }

  50% {
    transform: scale(1.2);
    background-color: rgb(255, 254, 106);
  }

  100% {
    transform: scale(1);
    background-color: rgb(255, 254, 106);
  }
}
</style>



