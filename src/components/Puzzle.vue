<template>
    <div class="wrapper" :style="{width:width+'px',height:height+'px'}">
        <div v-for="(item,index) in blockPoints" :key="item.id"
        :style="{
        width:blockWidth+'px',
        height:blockHeight+'px',
        left:item.x+'px',
        top:item.y+'px',
        backgroundImage:`url(${img})`,
        backgroundPosition:`-${correctPoints[index].x}px -${correctPoints[index].y}px`,
        opacity:index === blockPoints.length - 1 && 0,
        }"
        :ref="index===blockPoints.length-1? 'empty':'block'"
        @click="handleClick"
        :data-correctx="correctPoints[index].x"
        :data-correcty="correctPoints[index].y"
        ></div>
    </div>
</template>
<script>
export default {
    props:{
        img:{ //拼图图片地址
            type:String,
            required:true
        },
        row:{ //拼图的行数
            type:Number,
            required:true
        },
        col:{ //拼图的列数
            type:Number,
            required:true
        },
        width:{ //拼图的总宽
            type:Number,
            default:500
        },
        height:{ //拼图的总高
            type:Number,
            default:500
        }


    },
    computed:{
        blockWidth(){ //每个小方块的宽度
            return this.width/this.col;
        },
        blockHeight(){ //每个小方块的长度
            return this.height/this.row;
        },
        correctPoints(){ //存放小方块的最初位置，即正确位置（所有小方块的背景凑起来是一张完整的图）
            const{row,blockHeight,blockWidth}=this;//解构赋值
            const arr=[];//存放小方块最初位置的数组
            for(let i=0;i<row;i++){
                for(let j=0;j<row;j++){
                    arr.push({
                        x:j*blockWidth,//计算小方块的left值
                        y:i*blockHeight,//计算小方块的top值
                        id:new Date().getTime()+Math.random()*100//生成一个唯一的Id
                    })
                }
            }
            return arr;//返回这个数组
        },
        blockPoints(){
            //存放小方块乱序后的位置
            const correct=this.correctPoints;
            const length=correct.length;
            const lastEle=correct[length-1];//取出最后一块小方块 作为空白方块
            const newArr=[...correct];//用以拷贝数据的剩余运算符
            newArr.length=length-1;
            newArr.sort(()=>Math.random()-0.5);//将除最后一个方块以外的其他方块乱序 返回值有正有负
            newArr.push(lastEle);//需将最后一个方块加上
            return newArr;//返回乱序后的数组
        },
    },
    methods:{
        handleClick(e){ //控制方块移动的函数
        const blockDom=e.target;//获取事件对象
        const emptyDom=this.$refs.empty[0];//空白方块
        if(!this.checkBorder(blockDom,emptyDom)){ //调换位置的条件函数
        return;
        }
        const{left,top}=blockDom.style;//解构赋值
        blockDom.style.left=emptyDom.style.left;//将符合条件的有图方块与空白方块调换位置
        blockDom.style.top=emptyDom.style.top;
        emptyDom.style.left=left;
        emptyDom.style.top=top;
        const winFlag=this.checkWin();//检查是否通关的函数
        if(winFlag){ //游戏通关
            this.winGame(emptyDom);
        }
        },
        checkBorder(blockDom,emptyDom){ //判断是否能调换位置的函数
            const{left:domLeft,top:domTop}=blockDom.style;
            const{left:emptyLeft,top:emptyTop}=emptyDom.style;
            const xDis=Math.floor(Math.abs(parseFloat(domLeft)-parseFloat(emptyLeft)));
            const yDis=Math.floor(Math.abs(parseFloat(domTop)-parseFloat(emptyTop)));
            const flag=(domLeft===emptyLeft && yDis===parseInt(this.blockHeight)) || 
                  (domTop===emptyTop && xDis===parseInt(this.blockWidth));
            return flag;
        },
        checkWin(){  //检查是否通关的函数
            const domArr=this.$refs.block;//存放所有有图方块的一个数组
            return domArr.every(dom=>{ //确保每个有图方块都在正确的位置 
                const{left:domleft,top:domtop}=dom.style;
                const{correctx:correctX,correcty:correctY}=dom.dataset; //通过dataset拿到正确的位置
                const flag=parseInt(domleft)===parseInt(correctX) && parseInt(domtop)===parseInt(correctY);
                return flag;
            })
        },
        winGame(emptyDom){
            setTimeout(()=>{
                alert("恭喜你过关了");
                emptyDom.style.opacity=1;//最后一个方块的图片显示
                this.goNextLevel();
            },1000)
        },
        goNextLevel(){ //通向下一关的函数
           setTimeout(()=>{
           const answerFlag=window.confirm("要玩下一关吗？");
           if(answerFlag){
               this.$emit('nextLevel');//自定义事件，告知父组件
           }
            },3000)
        }

    }
}
</script>
<style scoped>
    .wrapper{
        border:1px solid black;
        position: relative;
    }
    .wrapper div{
        position: absolute;
        border:1px solid white;
        box-sizing: border-box;
        left:0;
        top:0;
    }
</style>