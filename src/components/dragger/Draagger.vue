<template>
    <div class="layout">
        <h1>This is define Dragger</h1>
        <div class="dragger-container">
            <div class="dragger-item" v-for="(item, index) in list" :key="index">
                <div 
                   class="dragger-item-handler"
                   @mousedown="(event)=>onDragBegin(event, {item, index, parent: list})"
                   @mousemove="onDragMove"
                   @mouseup="onDragEnd"
                >{{item.title}}</div>
                <div class="dragger-item-content">
                    <div class="dragger-item" v-for="(data, key) in item.children" :key="key">
                        <div 
                            class="dragger-item-handler"
                            @mousedown="(event)=>onDragBegin(event, {item:data, index:key, parent: item.children})"
                        >{{data.title}}</div>
                        <div class="dragger-item-content">

                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>

const { body }= document;

export default {
    name: 'dragger',
    data(){
        return {
            list: [],
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
            // console.log('[onDragBegin]', item, index, parent);
            const { target } = event;
            const parentDom = target.parentElement;
            const reactObj = parentDom.getBoundingClientRect();
            const { __dragOpts } = this;
            if(__dragOpts && __dragOpts.item !== item){
                this.__dragOpts = {
                    item,
                    index,
                    parent,
                    target: parentDom,
                    x: target.pageX - reactObj.left,
                    y: target.pageY - reactObj.top,
                    h: reactObj.height,
                    w: reactObj.width,
                    dragging: true,
                    createDom: true,
                };
            }else{
                this.__dragOpts = {
                    item,
                    index,
                    parent,
                    target: parentDom,
                    x: reactObj.left,
                    y: reactObj.top,
                    h: reactObj.height,
                    w: reactObj.width,
                    dragging: true,
                    createDom: true,
                };
            }
            console.log('[onDragBegin]', this.__dragOpts);
        },
        onDragMove(event){
            const { target } = event;
            const { __dragOpts } = this;
            if(__dragOpts && __dragOpts.dragging ){
                this.settMovingElement(event);
                console.log('[onDragMove]', event);
            }
        },
        onDragEnd(event){
            const { target } = event;
            const { __dragOpts } = this;
            if(__dragOpts){
                __dragOpts.dragging  = false;
                console.log('[onDragEnd]', event);
            }
        },
        onBodyMouseUp(){
            const { __dragOpts } = this;
            if(__dragOpts){
                __dragOpts.dragging  = false;
                this.__dom.hidden = true;
                console.log('[onBodyMouseUp]');
            }
        },
        onBodyMouseMove(event){
            const { __dragOpts } = this;
            if(__dragOpts && __dragOpts.dragging){
                console.log('[onBodyMouseMove]');
                this.settMovingElement(event);
            }
        },
        settMovingElement(event){
            const dom = this.__dom;
            const { __dragOpts } = this;
            const { target, x, y, w, h } = __dragOpts;
            const { pageX, pageY } = event;
            if(dom){
                dom.setAttribute('style', `
                    position: fixed;
                    z-index: 9999;
                    top: ${pageY}px;
                    left: ${pageX - x}px;
                    height: ${h}px;
                    width: ${w}px;
                `);
                dom.hidden = false;
            }else{
                console.log('[createMovingElement]', pageX, pageY, x, y);
                const dom = target.cloneNode(true);
                dom.id = 'dragger-body-move';
                dom.setAttribute('style', `
                    position: fixed;
                    z-index: 9999;
                    top: ${pageY}px;
                    left: ${pageX - x}px;
                    height
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
}
.dragger-container{
   
}

 .dragger-item{
       margin: 10px 0;
       border: 1px solid  #f2f2f2;
       background-color: #fff;
    }
    .dragger-item-handler{
        padding: 10px;
        cursor: move;
        user-select: none;
    }
</style>
