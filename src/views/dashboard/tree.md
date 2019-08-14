d3.tree()，创建一个树状图生成器
d3.tree().size()，定义树的大小
d3.hierarchy()，层级布局，需要和tree生成器一起使用，来得到绘制树所需要的节点数据和边数据

node.descendants() 得到所有的节点，已经经过转换的数据
node.links() 得到所有的边，已经经过转换的数据