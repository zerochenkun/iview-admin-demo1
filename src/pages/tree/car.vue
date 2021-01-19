<template>
  <div>
    <Tree :data="displayData" show-checkbox  expand-node @on-check-change="checkChange"></Tree>
  </div>
</template>
<script>
    import { cloneDeep } from 'lodash';
    const rootName = 'root';
    export default {
        name: 'asadsdas',
        components: {},
        props: {},
        data () {
            return {
                computedData: [],
                normalData: []
            }
        },
        computed: {
            displayData () {
                return this.computedData.concat(this.normalData);
            },
            displayData1 () {
                return [
                    {
                        title: 'parent 1',
                        expand: true,
                        selected: true,
                        children: [
                            {
                                title: 'parent 1-1',
                                expand: true,
                                children: [
                                    {
                                        title: 'leaf 1-1-1',
                                        disabled: true
                                    },
                                    {
                                        title: 'leaf 1-1-2'
                                    }
                                ]
                            },
                            {
                                title: 'parent 1-2',
                                expand: true,
                                children: [
                                    {
                                        title: 'leaf 1-2-1',
                                        checked: true
                                    },
                                    {
                                        title: 'leaf 1-2-1'
                                    }
                                ]
                            }
                        ]
                    }
                ]
            }
        },
        methods: {
            mergeChildren (node, defaultNode) {
                let children = node.children || [];
                let defaultChildren = defaultNode.children || [];
                return defaultChildren.map(item => {
                    let cItem = children.find(cur => cur.value === item.value);
                    if (cItem) return cItem;
                    return item;
                })
            },
            getNextChildren (node, children) {
                return node.children.filter(item => children.find(d => d.value === item.parent));
            },
            /**
             * 切换复选框状态
             * @param selectArr
             * @param node
             * @returns {boolean}
             */
            checkChange (selectArr, node) {
                if (node.singleFolder) return false;
                // 当选中的是根节点
                if (node.folder) {
                    let index = this.getIndexByNode(node);
                    if (index < 0) return false;
                    if (!this.defaultData[index + 1]) return false;
                    // 当添加
                    if (node.checked) {
                        let nextNode = this.computedData[index + 1];
                        let cur_children = node.children;
                        let defaultNextNode = cloneDeep(this.defaultData[index + 1]);
                        defaultNextNode.children = this.getNextChildren(defaultNextNode, cur_children);
                        // 已经存在则更新
                        if (nextNode) {
                            // 合并状态
                            nextNode.children = this.mergeChildren(nextNode, defaultNextNode);
                        } else {
                            // 展开下个节点
                            defaultNextNode.expand = true;
                            // 添加新的节点
                            this.computedData.push(defaultNextNode);
                        }
                    } else {
                        // 删除节点
                        this.computedData = this.computedData.slice(0, index + 1);
                    }
                } else {
                    // 处理叶子节点
                    let parentName = node.data.field;
                    // 找到父亲节点
                    let parentNode = this.computedData.find(item => item.value === parentName);
                    let index = this.getIndexByNode(parentNode);
                    if (!this.defaultData[index + 1]) return false;
                    let value = node.value;
                    let nextNode = this.computedData[index + 1];
                    let defaultNextNode = cloneDeep(this.defaultData[index + 1]);
                    // 可选儿子
                    let children = defaultNextNode.children.filter(item => item.parent === value);
                    // 当选中叶子节点
                    if (node.checked) {
                        if (nextNode) {
                            // 判断是否已经添加了
                            nextNode.children = nextNode.children.some(item => children.find(d => d.value === item.value)) ? nextNode.children : nextNode.children.concat(children);
                            // nextNode.children = nextNode.children.concat(children);
                            nextNode.checked = false;
                            nextNode.expand = true;
                        } else {
                            let newNode = defaultNextNode;
                            newNode.children = children;
                            newNode.expand = true;
                            this.computedData.push(newNode);
                        }
                    } else {
                        // 当取消节点
                        while (nextNode != null) {
                            // 显示节点
                            let showChildren = nextNode.children.filter(item => !children.some(d => d.value === item.value));
                            if (showChildren.length) {
                                nextNode.children = showChildren;
                            }
                            // 下一个节点有可能有联动关系
                            index++;
                            if (this.computedData[index + 1]) {
                                nextNode = this.computedData[index + 1];
                                defaultNextNode = cloneDeep(this.defaultData[index + 1]);
                                // 筛选联动儿子
                                children = defaultNextNode.children.filter(item => children.some(d => d.value === item.parent));
                            } else {
                                nextNode = null;
                            }
                            // 当前节点没有儿子则删除
                            if (!showChildren.length) {
                                this.computedData.splice(index, 1);
                                index--;
                            }
                        }
                    }
                }
            },
            // 获取当前节点的位置
            getIndexByNode (node) {
                for (let i = 0; i < this.defaultData.length; i++) {
                    if (node.value === this.defaultData[i].value) {
                        return i;
                    }
                }
                return -1;
            },
            // 排序
            sortTree () {
                this.defaultData = [];
                let finded = true;
                // 当前的Name
                let parent = rootName;
                while (parent && finded) {
                    let node = this.mapTree[parent];
                    if (!node) {
                        finded = false;
                    } else {
                        this.defaultData.push(node);
                        parent = node.value;
                    }
                }
                return this.defaultData;
            }
        },
        created () {
            // 获取初始化数据
            const data = { 'data': [{ 'parent': 'root', 'children': [{ 'data': { 'field': 'cars' }, 'title': '悦程', 'value': 'yuecheng' }, { 'data': { 'field': 'cars' }, 'title': '逸动', 'value': 'yidong' }, { 'data': { 'field': 'cars' }, 'title': '奔奔mini', 'value': 'A101_A102' }, { 'data': { 'field': 'cars' }, 'title': '杰勋', 'value': 'CV11' }, { 'data': { 'field': 'cars' }, 'title': 'CX30三厢', 'value': 'C118' }], 'title': '车系', 'value': 'cars' }, { 'parent': 'cars', 'children': [{ 'parent': 'yuecheng', 'data': { 'field': 'model' }, 'title': '手动型', 'value': 'manual' }, { 'parent': 'yidong', 'data': { 'field': 'model' }, 'title': '自动型', 'value': 'auto' }, { 'parent': 'A101_A102', 'data': { 'field': 'model' }, 'title': '1.0L IMT 国四', 'value': 'SC7106B4' }, { 'parent': 'A101_A102', 'data': { 'field': 'model' }, 'title': '1.0L 5MT 国四', 'value': 'SC7106A4' }, { 'parent': 'CV11', 'data': { 'field': 'model' }, 'title': '2.0L 4AT 国三', 'value': 'SC6442A' }, { 'parent': 'CV11', 'data': { 'field': 'model' }, 'title': '2.0L 4AT 国四', 'value': 'SC6442A4' }, { 'parent': 'CV11', 'data': { 'field': 'model' }, 'title': '1.5L 5MT 国四', 'value': 'SC7152' }, { 'parent': 'C118', 'data': { 'field': 'model' }, 'title': '1.6L 5MT 国四', 'value': 'SC7163B4' }, { 'parent': 'C118', 'data': { 'field': 'model' }, 'title': '2.0L 4AT 国四_2', 'value': 'SC7200D4' }, { 'parent': 'C118', 'data': { 'field': 'model' }, 'title': '2.0L 4AT 国三_2', 'value': 'SC7200D' }], 'title': '车型', 'value': 'model' }, { 'parent': 'model', 'children': [{ 'parent': 'manual', 'data': { 'field': 'modelcode' }, 'title': 'M01', 'value': 'M01' }, { 'parent': 'auto', 'data': { 'field': 'modelcode' }, 'title': 'A01', 'value': 'A01' }, { 'parent': 'SC7106B4', 'data': { 'field': 'modelcode' }, 'title': 'IMT 豪华型', 'value': 'SC7106B4.A42A' }, { 'parent': 'SC7106B4', 'data': { 'field': 'modelcode' }, 'title': 'IMT 舒适型', 'value': 'SC7106B4.A31A' }, { 'parent': 'SC7106B4', 'data': { 'field': 'modelcode' }, 'title': 'SC7106B4.A4', 'value': 'SC7106B4.A4' }, { 'parent': 'SC7106A4', 'data': { 'field': 'modelcode' }, 'title': '舒适型', 'value': 'SC7106A4.A32A' }, { 'parent': 'SC7106A4', 'data': { 'field': 'modelcode' }, 'title': 'SC7106A4.A2', 'value': 'SC7106A4.A2' }, { 'parent': 'SC7106A4', 'data': { 'field': 'modelcode' }, 'title': '标准型', 'value': 'SC7106A4.A21A' }, { 'parent': 'SC6442A', 'data': { 'field': 'modelcode' }, 'title': '2007款 舒适性 5座', 'value': 'SC6442A.D3A' }, { 'parent': 'SC6442A', 'data': { 'field': 'modelcode' }, 'title': '2007款 豪华型 5座', 'value': 'SC6442A.D4A' }, { 'parent': 'SC6442A', 'data': { 'field': 'modelcode' }, 'title': '2007款 超豪华型 5座', 'value': 'SC6442A.D5A' }, { 'parent': 'SC6442A4', 'data': { 'field': 'modelcode' }, 'title': '2008款 超豪华型 5座', 'value': 'SC6442A4.D5A' }, { 'parent': 'SC7152', 'data': { 'field': 'modelcode' }, 'title': '2009款 舒适性 5座', 'value': 'SC7152.DAA' }, { 'parent': 'SC7152', 'data': { 'field': 'modelcode' }, 'title': 'SC7152.E2A', 'value': 'SC7152.E2A' }, { 'parent': 'SC7163B4', 'data': { 'field': 'modelcode' }, 'title': '- - 5座', 'value': 'SC7163B4.C4' }, { 'parent': 'SC7163B4', 'data': { 'field': 'modelcode' }, 'title': '- - 5座_2', 'value': 'SC7163B4.C3' }, { 'parent': 'SC7163B4', 'data': { 'field': 'modelcode' }, 'title': 'CX30三厢国四1.6L标准型MT', 'value': 'SC7163B4.C2' }, { 'parent': 'SC7200D4', 'data': { 'field': 'modelcode' }, 'title': '- - 5座_3', 'value': 'SC7200D4.C4' }, { 'parent': 'SC7200D4', 'data': { 'field': 'modelcode' }, 'title': '- - 5座_4', 'value': 'SC7200D4.C3' }, { 'parent': 'SC7200D', 'data': { 'field': 'modelcode' }, 'title': '- - 5座_5', 'value': 'SC7200D.C4' }, { 'parent': 'SC7200D', 'data': { 'field': 'modelcode' }, 'title': 'SC7200D.C3', 'value': 'SC7200D.C3' }], 'title': '车型配置代码', 'value': 'modelcode' }, { 'children': [{ 'data': { 'field': 'EMUIVersion' }, 'title': '发版', 'value': 'pubY' }, { 'data': { 'field': 'EMUIVersion' }, 'title': '不发版', 'value': 'pubN' }], 'title': '发版状态', 'value': 'EMUIVersion' }], 'success': true }
            const treeData = data.data;
            this.mapTree = {};
            treeData.forEach(item => {
                if (item.parent) {
                    item.folder = true;
                    this.mapTree[item.parent] = item;
                } else {
                    item.singleFolder = true;
                    if (item.children && item.children.length) {
                        item.children.forEach(item => {
                            item.singleFolder = true;
                        }
                        );
                    }
                    this.normalData.push(item);
                }
            })
            // 排序 处理数据不一定是有序数据
            this.sortTree();
            console.log('defaultData', this.defaultData);
            this.computedData = [this.defaultData[0]];
        }

    }
</script>
<style scoped>
</style>
