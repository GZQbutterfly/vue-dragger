<template>
    <div class="layout" ref="layout">
        <h1>This is define Dragger</h1>
        <div class="dragger-container">
            <transition-group>
                <div class="dragger-item card-list" v-for="(item, index) in list" :key="index">
                    <!--  -->
                    <template v-if="dragOpts.dragging === 1 && dragOpts.item === item">
                        <div
                            class="dragger-item-mirror"
                            :style="{
                                width: dragOpts.w + 'px',
                                height: dragOpts.h + 'px',
                            }"
                        >
                        </div>
                    </template>

                    <template v-else>
                        <div
                            class="dragger-item-handler"
                            @mousedown="onDragBegin($event, { item, index })"
                            @mousemove="onDragMove"
                            @mouseup="onDragEnd"
                            @mouseenter="onMouseEnter($event, { item, index })"
                        >
                            {{item.title}}
                        </div>
                        <div class="dragger-item-content">
                            <transition-group>
                                <div 
                                    class="dragger-item card-item" 
                                    v-for="(data, key) in item.children" 
                                    :key="key"
                                >
                                    <template v-if="dragOpts.dragging === 1 && dragOpts.item === data">
                                        <div
                                            class="dragger-item-mirror"
                                            :style="{
                                                width: dragOpts.w + 'px',
                                                height: dragOpts.h + 'px',
                                            }"
                                        >
                                        </div>
                                    </template>
                                    <template v-else>
                                        <div 
                                            class="dragger-item-handler"
                                            @mousedown="onDragBegin($event, { item: data, index: key, parent: item.children})"
                                            @mousemove="onDragMove"
                                            @mouseup="onDragEnd"
                                            @mouseenter.stop="onMouseEnter($event, { item: data, index: key, parent: item.children})"
                                        >{{data.title}}</div>
                                        <div class="dragger-item-content">

                                        </div>
                                    </template>
                                </div>
                            </transition-group>
                        </div>
                    </template>
                </div>
            </transition-group>
        </div>
    </div>
</template>
<script>
import { differenceWith, isEqual, debounce  } from 'lodash';
const { body }= document;

function diff(obj1, obj2){
    if(obj1 === undefined && obj2 === undefined){
        return true;
    }
    if(obj1 === undefined || obj2 === undefined){
        return false;
    }
    return !differenceWith(obj1, obj2, isEqual).length;
}

function getOffsetTop(el){
    const { offsetParent, offsetTop } = el;
    return offsetParent ? offsetTop + getOffsetTop(offsetParent): offsetTop;
}

function getOffsetLeft(el){
    const { offsetParent, offsetLeft } = el;
    return offsetParent ? offsetLeft + getOffsetLeft(offsetParent): offsetLeft;
}


function findItemEndRemove(list, item){
    for(let i = 0; i< list.length; i++){
        const _item = list[i];
        if(_item.children && _item.children.length){
            for(let j = 0; j< _item.children.length; j++){
                if(diff(_item.children[j], item)){
                    _item.children.splice(j, 1);
                    j--;
                }
            }
        }else{
            if(diff(_item, item)){
                list.splice(i, 1);
                i--;
            }
        }
    }
}

export default {
    name: 'dragger',
    data(){
        return {
            list: [],
            dragOpts: {
                item: {},
                dragItem: {},
                index: -1,
                parent: {},
                target: null,
                x: 0,
                y: 0,
                h: 0,
                w: 0,
                dragging: -1, // -1 结束； 0 开始； 1 进行中
            }
        };
    },
    mounted(){
        const list = [];
        for(let i=0; i < 10; i++){
            list.push({
                title: 'Dragger aa' + i,
                children: [
                    { title: 'Dragger aa children ' + i + '001' },
                    { title: 'Dragger aa children ' + i + '002' }
                ]
            });
        }
        this.list = list;
        body.addEventListener('mouseup', this.onBodyMouseUp);
        body.addEventListener('mousemove', this.onBodyMouseMove);
    },
    beforeDestroy(){
        body.removeEventListener('mouseup', this.onBodyMouseUp);
        body.removeEventListener('mousemove', this.onBodyMouseMove);
    },
    methods:{
        onDragBegin(event, { item, index, parent }){
            const { target } = event;
            const parentDom = target.parentElement;
            const { dragOpts } = this;
            if(dragOpts && dragOpts.dragItem !== item){
                if(this.__dom){
                    body.removeChild(this.__dom);
                    this.__dom = null;
                }
            }
            this.dragOpts = {
                item,
                index,
                parent,
                target: parentDom,
                x: event.pageX - getOffsetLeft(parentDom),
                y: event.pageY - getOffsetTop(parentDom) + this.$refs.layout.scrollTop,
                h: parentDom.offsetHeight,
                w: parentDom.offsetWidth,
                dragging: 0,
            };    
            window.clearTimeout(this.__timer);
            this.__timer = null;
            // console.table('[onDragBegin]', this.dragOpts.dragging);
        },
        onDragMove(event){
            const { target } = event;
            const { dragOpts } = this;
            if(dragOpts &&  dragOpts.dragging === 0){
                if(this.__timer){
                    dragOpts.dragging = 1;
                    // console.log('延迟操作 moving');
                    this.settMovingElement(event);
                    // console.log('[onDragMove]', event);
                    this.__timer = null;
                }else{
                    this.__timer = setTimeout(()=>{}, 50);
                }
            }
        },
        onMouseEnter(event, { item, index, parent }){
            const { dragOpts } = this;
            if(dragOpts && dragOpts.dragging === 1){
                const { item: dragItem, index: dragIndex } = dragOpts;
                const newList = [...this.list];
                if(parent){
                   const endIndex = newList.findIndex(ele => diff(ele.children, parent));
                   const { children } = newList[endIndex];
                    if(dragOpts.parent){
                        if(dragOpts.parent === parent){
                            // 同父级中拖拽
                            children[index] = dragItem;
                            children[dragIndex] = item;
                            dragOpts.index = index;
                        }else{
                            // 不同父级拖拽 从A脱离进入B
                            const startIndex = newList.findIndex(ele => diff(ele , dragItem));
                            // 移除原来
                            newList[startIndex].children.splice(dragIndex, 1);
                            // 添加新的位置
                             if(index === parent.length -1){
                                // end
                                children[index + 1] = dragItem;
                                dragOpts.index = index + 1;
                            }else{
                                // start
                                newList[endIndex].children = [dragItem, ...children];
                                dragOpts.index = 0;
                            }
                            dragOpts.parent = children;
                            console.log('[跨集拖拽]');
                        }
                    }else{
                        // 从顶级移入子集
                        newList.splice(dragIndex, 1);
                        if(index === parent.length -1){
                            // end
                            children[index + 1] = dragItem;
                            dragOpts.index = index + 1;
                        }else{
                            // start
                            newList[endIndex].children = [dragItem, ...children];
                            dragOpts.index = 0;
                        }
                        dragOpts.parent = newList[endIndex].children;
                        console.log('[跨集拖拽 no dragOpts.parent]');
                    }
                }else{
                    if(dragOpts.parent){
                        // 子集移入顶级
                        const startIndex = newList.findIndex(ele => diff(ele.children, dragOpts.parent));
                        newList[startIndex].children.splice(dragIndex, 1);
                        if(index){
                            newList.splice(index, 0, dragItem);
                            dragOpts.index = index;
                        }else{
                            newList.unshift(dragItem);
                            dragOpts.index = 0;
                        }
                        console.log('[子集拖拽 dragOpts.parent]');
                    }else{
                        // 顶级之间拖拽
                        newList[index] = dragItem;
                        newList[dragIndex] = item;
                        dragOpts.index = index;
                    }
                    dragOpts.parent = null;
                }
                this.list = newList;
                console.log('[onMouseEnter]');
            }
        },
        onDragEnd(event){
            const { target } = event;
            const { dragOpts } = this;
            if(dragOpts){
                dragOpts.dragging  = -1;
                // console.log('[onDragEnd]', event);
                window.clearTimeout(this.__timer);
                this.__timer = null;
            }
        },
        onBodyMouseUp(){
            const { dragOpts } = this;
            if(dragOpts){
                dragOpts.dragging  = -1;
                if(this.__dom){
                    this.__dom.hidden = true;
                }
                // console.log('[onBodyMouseUp]');
            }
        },
        onBodyMouseMove(event){
            const { dragOpts } = this;
            if(dragOpts && dragOpts.dragging === 1){
                // console.log('[onBodyMouseMove]');
                this.settMovingElement(event);
            }
        },
        settMovingElement(event){
            const dom = this.__dom;
            const { dragOpts } = this;
            const { target, x, y, w, h } = dragOpts;
            const { pageX, pageY } = event;
            if(dom){
                dom.setAttribute('style', `
                    position: fixed;
                    z-index: 9999;
                    top: ${pageY - y}px;
                    left: ${pageX - x}px;
                    height: ${h}px;
                    width: ${w}px;
                    pointer-events: none;
                `);
                dom.hidden = false;
            }else{
                const dom = target.cloneNode(true);
                dom.id = 'dragger-body-move';
                dom.setAttribute('style', `
                    position: fixed;
                    z-index: 9999;
                    top: ${pageY - y}px;
                    left: ${pageX - x}px;
                    height: ${h}px;
                    width: ${w}px;
                    pointer-events: none;
                `);
                document.body.appendChild(dom);
                this.__dom = dom;
            } 
        }
    }
}
</script>
<style lang="less">
.layout{
    width: 500px;
    height: 700px;
    margin: 0 auto;
    border: 1px solid #454545;
    overflow: auto;
    padding: 10px;
}
.dragger-container{
   .dragger-item-mirror{
       border: 1px dashed #333;
       background-color: #555;
   }
}

.dragger-item{
    margin: 10px 0;
    border: 1px solid  #f2f2f2;
    background-color: #fff;
}
.dragger-item-handler{
    padding: 10px;
    // cursor: move;
    user-select: none;
}

.card-list{
    border: 1px solid blue;
    padding: 10px;
}

.card-item{
    border: 1px solid red;
}
</style>
