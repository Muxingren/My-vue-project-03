<template>
    <div>
        <Puzzle v-bind="puzzleData[index]" @nextLevel="nextLevel"></Puzzle>
    </div>
</template>
<script>
import Puzzle from './components/Puzzle.vue';
//导入子组件
export default {
    data(){
        return{
            index:0,
            puzzleData:[ //存放拼图数据
                {col:3,row:3,img:'../images/1.jpeg'},
                {col:4,row:4,img:'../images/2.jpg'},
                {col:5,row:5,img:'../images/3.jpg'},
            ]
        }
    },
    methods:{
        nextLevel(){ //子组件的自定义事件 下一关
            this.index ++;
            if(this.index>this.puzzleData.length-1){
               const newFlag= window.confirm("你已经通关了，需要再来一轮吗？");
               if(newFlag){
                   this.index=0;//回到第一关开始
               }
            }
        }
    },
    components:{
        Puzzle,//组件的注册
    }
}
</script>