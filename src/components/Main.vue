<template>
  <div class="container">
    <div class="reg-container">
      <div class="outline">Rimulator</div> 
      <hr>
      <div class="reg-scroller">
        <table class="reg-table">
          <tbody v-for="(R,i) in this.Reg" v-bind:key="i" >
            <tr style="width:100%" class="reg-line">
              <td class="reg-name">{{R.name}}</td>
              <td class="reg-value">{{this.CalcRadix(R.value,this.RegRadix)}}</td>
            </tr>   
            <tr class="hr"></tr>
          </tbody>
        </table>
      </div> 
    </div>
    <div class="code-container" style="overflow:hidden;">
      <table style="padding-left:20px;width:600px;">
        <tr class="title" >
          <td style="width:150px;">机器码</td>
          <td style="width:150px;">预处理</td>
          <td style="width:150px;">Code</td>
        </tr>
      </table>
      <hr>
      <div class="code-scroller">
        <table class="code-table">   
          <tbody v-for="(c,i) in this.Code" v-bind:key="i"
            v-bind:class="{'code-line':true,'breakpoint-run':c.run&&c.breakpoint,'run':c.run,'breakpoint':c.breakpoint,'active':c.active}"
              v-on:click.left="MoveFocus(i)"
                v-on:click.right="ChangeBP(i)"
                  v-on:keydown.enter="NewCode(i)"
                    v-on:keydown.up="Up(i)"
                      v-on:keydown.down="Down(i)"
                        v-on:keydown.delete="Remove(i)">
            <tr>
              <td class="code-linenumber">
              {{CalcLinenumber(i)}}
              </td>
              <td class="code-mc">
                {{CalcRadix(c.mc,16)}}
              </td>
              <td class="code-pre">
                <span class="code-op">{{c.op}}</span>
                <span v-if="c.tag=='E'">
                  <span class="code-rs">--</span>
                  <span class="code-rs">--</span>
                  <span class="code-rs">--</span>
                </span>
                <span v-if="c.tag!='E'">
                  <span v-if="this.Inst[c.op].type=='r'">
                    <span class="code-rs">{{this.Reg[c.rd].name}}</span>
                    <span class="code-rs">{{this.Reg[c.rs1].name}}</span>
                    <span class="code-rs">{{this.Reg[c.rs2].name}}</span>
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='i'">
                    <span class="code-rs">{{this.Reg[c.rd].name}}</span>
                    <span class="code-rs">{{this.Reg[c.rs1].name}}</span>
                    <span class="code-rs">{{c.rs2}}</span>
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='jr'">
                    <span class="code-rs">{{this.Reg[c.rd].name}}</span>
                    <span class="code-rs">{{c.rs2}}({{this.Reg[c.rs1].name}})</span>
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='l'">
                    <span class="code-rs">{{this.Reg[c.rd].name}}</span>
                    <span class="code-rs">{{c.rs2}}({{this.Reg[c.rs1].name}})</span>
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='s'">
                    <span class="code-rs">{{this.Reg[c.rs2].name}}</span>
                    <span class="code-rs">{{c.rd}}({{this.Reg[c.rs1].name}})</span>
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='b'">
                    <span class="code-rs">{{this.Reg[c.rs1].name}}</span>
                    <span class="code-rs">{{this.Reg[c.rs2].name}}</span>
                    <span class="code-rs">{{c.rd}}</span>
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='j'">
                    <span class="code-rs">{{this.Reg[c.rd].name}}</span>
                    <span class="code-rs">{{c.rs1}}</span>
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='u'">
                    <span class="code-rs">{{this.Reg[c.rd].name}}</span>
                    <span class="code-rs">{{c.rs1}}</span>
                  </span>
                  <span v-else-if="c.op=='mret'">
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='sys'&&c.op[c.op.length-1]!='i'">
                    <span class="code-rs">{{this.Reg[c.rd].name}}</span>
                    <span class="code-rs">{{this.Reg[c.rs2].name}}</span>
                    <span class="code-rs">{{this.Reg[c.rs1].name}}</span>
                  </span>
                  <span v-else-if="this.Inst[c.op].type=='sys'">
                    <span class="code-rs">{{this.Reg[c.rd].name}}</span>
                    <span class="code-rs">{{this.Reg[c.rs2].name}}</span>
                    <span class="code-rs">{{c.rs1}}</span>
                  </span>
                  
                  <span v-else-if="this.Inst[c.op].type=='n'">
                    <span class="code-rs">--</span>
                    <span class="code-rs">--</span>
                    <span class="code-rs">--</span>
                  </span>                            
                </span>
              </td>
              <td v-html="c.or" class="code-or" v-bind:contenteditable="c.active" 
                v-bind:id="i" v-on:input="Compile(i)">
              </td>
            </tr>
            <tr class="hr"></tr>
          </tbody>
        </table>
      </div>
      <div class="code-out" v-html="Output" 
        v-bind:contenteditable="this.ConsoleModel=='edit'"
        v-on:input="EditCode">
      </div>
      <div>
        <button class="nice-button" v-on:click="Reset">重置</button>
        <button class="nice-button" v-on:click="Step">单步</button>
        <button class="nice-button">中断</button>          
        <button class="nice-button" v-on:click="ShowMachine"
          v-bind:class="{'nice-button':true,'active':this.ConsoleModel=='machine'}" >
          译码
        </button>
        <button class="nice-button" v-on:click="ShowMessage"
          v-bind:class="{'nice-button':true,'active':this.ConsoleModel=='message'}" >
          消息
        </button>
        <button class="nice-button" v-on:click="Edit"
          v-bind:class="{'nice-button':true,'active':this.ConsoleModel=='edit'}" >
          导入
        </button>
        <button class="nice-button">链接</button>
      </div>
    </div>
    <div class="stack-container">
      <div style="margin-left:15%;margin-bottom:3%">
        <button v-bind:class="{'nice-button':true,'active':this.DataModel=='text'}" 
          v-on:click="this.DataModel='text'">text</button>
        <button v-bind:class="{'nice-button':true,'active':this.DataModel=='data'}" 
          v-on:click="this.DataModel='data'">data</button>
        <button v-bind:class="{'nice-button':true,'active':this.DataModel=='stack'}" 
          v-on:click="this.DataModel='stack'">stack</button>
      </div>
      <hr>
      <div v-if="this.DataModel=='stack'" class="stack-switch">   
        <div class="stack-scroller">
          <table class="stack-table">
            <tbody v-for="(s,i) in this.Stack" v-bind:key="i" class="stack-line"> 
              <tr>
                <td class="stack-address">{{this.CalcRadix(this.StackAddr(i),16)}}</td>
                <td class="stack-value">{{this.CalcRadix(s,this.StackRadix)}}</td> 
              </tr>
              <tr class="hr"></tr>
            </tbody>
          </table>
        </div>
      </div>
      <div v-if="this.DataModel=='data'" class="stack-switch">
        <div class="stack-scroller">
          <table class="stack-table">
            <tbody v-for="(s,i) in this.Data" v-bind:key="i" class="stack-line"> 
              <tr>
                <td class="stack-address">{{this.CalcRadix(this.DataAddr(i),16)}}</td>
                <td class="stack-value">{{this.CalcRadix(s,this.StackRadix)}}</td> 
              </tr>
              <tr class="hr"></tr>
            </tbody>
          </table>
        </div>
      </div>
      <div v-if="this.DataModel=='text'" class="stack-switch">
        <div class="stack-scroller">
          <table class="stack-table">
            <tbody v-for="(c,i) in this.Code" v-bind:key="i" class="stack-line"> 
              <tr>
                <td class="stack-address">{{this.CalcRadix(this.TextAddr(i),16)}}</td>
                <td class="stack-value">{{this.CalcRadix(c.mc,this.StackRadix)}}</td> 
              </tr>
              <tr class="hr"></tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  updated:function(){      
    this.$nextTick(function () {
    // 仅在渲染整个视图之后运行的代码
      if(this.Focus!=null){
        document.getElementById(String(this.Focus)).focus();
      }
    })
  },
  data:function(){
    return {
      //寄存器相关
      pc:-1,
      Reg: [
        {name:"zero",value:0},
        {name:"ra",value:0},
        {name:"sp",value:0xbffffff0},
        {name:"gp",value:0},
        {name:"tp",value:0},
        {name:"t0",value:0},
        {name:"t1",value:0},
        {name:"t2",value:0},
        {name:"s0",value:0},
        {name:"s1",value:0},
        {name:"a0",value:0},
        {name:"a1",value:0},
        {name:"a2",value:0},
        {name:"a3",value:0},
        {name:"a4",value:0},
        {name:"a5",value:0},
        {name:"a6",value:0},
        {name:"a7",value:0},
        {name:"s2",value:0},
        {name:"s3",value:0},
        {name:"s4",value:0},
        {name:"s5",value:0},
        {name:"s6",value:0},
        {name:"s7",value:0},
        {name:"s8",value:0},
        {name:"s9",value:0},
        {name:"s10",value:0},
        {name:"s11",value:0},
        {name:"t3",value:0},
        {name:"t4",value:0},
        {name:"t5",value:0},
        {name:"t6",value:0},
        {name:"mtvec",value:0},
        {name:"mepc",value:0},
        {name:"mcause",value:0},
        {name:"mie",value:0},
        {name:"mip",value:0},
        {name:"mtval",value:0},
        {name:"mscratch",value:0},
        {name:"mstatus",value:0},
      ],
      //code相关
      Focus: null,
      Code:[
        {or:"addi x1,x1,4",op:"addi",rd:1,rs1:1,rs2:4, mc:0x00408093, active:false,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        // {or:"beq x1,x1,xxx",op:"addi",rd:1,rs1:1,rs2:4, mc:0, active:false,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        // {or:"jal x1,xxx",op:"jal",rd:1,rs1:1,rs2:1, mc:0, active:false,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        // {or:"addi x1,x1,4",op:"addi",rd:1,rs1:1,rs2:4, mc:0, active:false,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        // {or:"addi x1,x1,4",op:"addi",rd:1,rs1:1,rs2:4, mc:0, active:false,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        // {or:"xxx:addi x1,x1,4",op:"addi",rd:1,rs1:1,rs2:4, mc:0, active:true,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        // {or:"jalr x0,0(x1)",op:"jalr",rd:1,rs1:1,rs2:1, mc:0, active:false,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        // {or:"sw sp,0(x1)",op:"add",rd:0,rs1:1,rs2:1, mc:0, active:false,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        // {or:"lw x1,0(x1)",op:"lw",rd:0,rs1:1,rs2:3, mc:0, active:false, run:false,breakpoint:false, tag:'I', Message:"无错误"},
      ],
      Inst:{
        'lui':    {op7:"0110111",type:'u'},
        'auipc':  {op7:"0010111",type:'u'},
        'jal':    {op7:"1101111",fun3:"000",type:'j'},
        'jalr':   {op7:"1100111",fun3:"000",type:'jr'},
        'beq':    {op7:"1100011",fun3:"000",type:'b'},
        'bne':    {op7:"1100011",fun3:"001",type:'b'},
        'blt':    {op7:"1100011",fun3:"100",type:'b'},
        'bge':    {op7:"1100011",fun3:"101",type:'b'},
        'bltu':   {op7:"1100011",fun3:"110",type:'b'},
        'bgeu':   {op7:"1100011",fun3:"111",type:'b'},
        'lb':     {op7:"0000011",fun3:"000",type:'l'},
        'lh':     {op7:"0000011",fun3:"001",type:'l'},
        'lw':     {op7:"0000011",fun3:"010",type:'l'},
        'lbu':    {op7:"0000011",fun3:"100",type:'l'},
        'lhu':    {op7:"0000011",fun3:"101",type:'l'},
        'sb':     {op7:"0100011",fun3:"000",type:'s'},
        'sh':     {op7:"0100011",fun3:"001",type:'s'},
        'sw':     {op7:"0100011",fun3:"010",type:'s'},
        'addi':   {op7:"0010011",fun3:"000",type:'i'},
        'slti':   {op7:"0010011",fun3:"010",type:'i'},
        'sltiu':  {op7:"0010011",fun3:"011",type:'i'},
        'xori':   {op7:"0010011",fun3:"100",type:'i'},
        'ori':    {op7:"0010011",fun3:"110",type:'i'},
        'andi':   {op7:"0010011",fun3:"111",type:'i'},
        'slli':   {op7:"0010011",fun7:"00000000",fun3:"001",type:'i'},
        'srli':   {op7:"0010011",fun7:"00000000",fun3:"101",type:'i'},
        'srai':   {op7:"0010011",fun7:"01000000",fun3:"101",type:'i'},
        'add':    {op7:"0110011",fun7:"00000000",fun3:"000",type:'r'},
        'sub':    {op7:"0110011",fun7:"01000000",fun3:"000",type:'r'},
        'sll':    {op7:"0110011",fun7:"00000000",fun3:"001",type:'r'},
        'slt':    {op7:"0110011",fun7:"01000000",fun3:"010",type:'r'},
        'sltu':   {op7:"0110011",fun7:"00000000",fun3:"011",type:'r'},
        'xor':    {op7:"0110011",fun7:"00000000",fun3:"100",type:'r'},
        'srl':    {op7:"0110011",fun7:"00000000",fun3:"100",type:'r'},
        'sra':    {op7:"0110011",fun7:"00000000",fun3:"101",type:'r'},
        'or':     {op7:"0110011",fun7:"01000000",fun3:"101",type:'r'},
        'and':    {op7:"0110011",fun7:"00000000",fun3:"110",type:'r'},
        'fence':  {op7:"0001111",type:'f'},
        'fence.i':{op7:"0001111",type:'f'},
        'ecall':  {op7:"1110011",type:'e'},
        'ebreak': {op7:"1110011",type:'e'},
        'csrrw':  {op7:"1110011",fun3:"001",type:'sys'},
        'csrrs':  {op7:"1110011",fun3:"010",type:'sys'},
        'csrrc':  {op7:"1110011",fun3:"011",type:'sys'},
        'csrrwi': {op7:"1110011",fun3:"101",type:'sys'},
        'cssrrsi':{op7:"1110011",fun3:"110",type:'sys'},
        'csrrci': {op7:"1110011",fun3:"111",type:'sys'},
        'mret':   {mc:"00110000001000000000000001110011",type:'sys'},
        '---':    {type:'n'},
      },
      Label:{},
      //栈相关
      Stack:[0,0,0],
      Data:[0,0,0],
      //格式相关
      RegRadix:16,
      StackRadix:16,
      MCRadix:16,
      DataModel:'stack',
      ConsoleModel:'message',
    }
  },
  methods:{
    CalcLinenumber:function(i){
      var n=String(this.Code.length).length
      return String(i).padStart(n,'0')
    },
    FreshAll:function(){
      for(var j=0;j<this.Code.length;j++){
        this.Compile(j);
      }
    },
    Vaild:function(s){
      　return RegExp(/\w/g).test(s) 
    },
    EditCode:function(){
      if(this.ConsoleModel!='edit'){
        return;
      }
      var str=document.getElementsByClassName('code-out')[0].innerHTML;
      str=str.replace(/&nbsp;/g,' ').replace(/<div[^>]*>|"|<span[^>]*>|<\/span>/g,' ').replace(/<\/div>/g,'<br>');
      str=str.split('<br>');
      for(var i in str){
        str[i]=str[i].replace(/#.*/g,'')
      }
      str=str.filter(this.Vaild);
      this.Code=[];
      for(var i=0;i<str.length;i++){
        this.Code.push(
          {or:str[i],op:"---",rd:0,rs1:0,rs2:0, mc:0x000000, active:false,run:false,breakpoint:false, tag:'I', Message:"无错误"},
        )
      }
      if(this.Code.length==0){
        this.Code.splice(i,1, {or:"",op:"--",rd:0,rs1:0,rs2:0, mc:0, active:false,run:false,breakpoint:false, tag:'E', Message:"空指令"})
      }
    },
    CalcRadix:function(v, radix){
      if(radix==16){
        if(v=='-') return "------ ----"
        return "0x"+("0000"+((v>>16)&0xffff).toString(16)).slice(-4)
                  +" "+("0000"+(v&0xffff).toString(16)).slice(-4);
      }
    },
    Sext:function(imm,len){
      var ret=imm;
      if(imm>>(len-1)){
        ret=0xffffffff;
        ret=ret<<len;
        ret+=imm;
      }
      return ret;
    },
    MoveFocus(i){
      /*
        逻辑:移动焦点到i
        首先检查当前焦点和所有错误语句, 然后更新当前焦点内容
        最后修改焦点. 总是默认为innerHTML是最新的，但是现在却是Code是最新的。
      */
      if(this.Focus!=null){
        this.Code[this.Focus].active=false;
        this.Code[this.Focus].or=document.getElementById(String(this.Focus)).innerHTML;
      }
      if(i!=null){
        this.Code[i].active=true;
      } 
      this.Focus=i;
    },
    NewCode:function(i){
      var e=window.event;
      e.returnValue=false;
      var x=window.getSelection().getRangeAt(0).endOffset;
      var t=document.getElementById(String(this.Focus)).innerHTML;
      var keys=Object.keys(this.Label)
      if(x!=0||t==""){
        this.Code.splice(i+1,0,{or:"",op:'---',rs1:0,rs2:0,rd:0,mc:0,active:false,run:false,breakpoint:false,tag:'E',Message:'空指令'}); 
        for(var j in keys){
          //所有大于i的行,行数+1
          if(this.Label[keys[j]]>i){
            this.Label[keys[j]]+=1;
          }
        }
        this.MoveFocus(i+1);
      }
      else{
        this.Code[i].active=false;
        this.Code.splice(i,0,{or:"",op:'---',rs1:0,rs2:0,rd:0,mc:0,active:false,run:false,breakpoint:false,tag:'E',Message:'空指令'});
        document.getElementById(String(i)).innerHTML="";//焦点移动前默认网页中是最新的
        for(var j in keys){
          //所有大于等于i的行,行数+1
          if(this.Label[keys[j]]>=i){
            this.Label[keys[j]]+=1;
          }
        } 
        this.MoveFocus(i);
      }
      this.FreshAll() 
    },
    Remove:function(i){
      var e=window.event;
      if(e.keyCode==8) return;
      e.returnValue=false;
      //焦点移到i-1行，如果是0行，焦点不变
      if(i>=1){
        this.MoveFocus(i-1);
      }
      //删除标签，如果存在的话
      if(Object.values(this.Label).indexOf(i)!=-1){
        var keys=Object.keys(this.Label);
        for(var j in keys){
          if(this.Label[keys[j]]==i){
            delete this.Label[keys[j]];
          }
          else if(this.Label[key]>i){
            this.Label[key]-=1;
          }
        }
      }
      if(this.Code.length>1){
        this.Code.splice(i,1);
      }
      else{
        this.Code.splice(i,1, {or:"",op:"--",rd:0,rs1:0,rs2:0, mc:0, active:false,run:false,breakpoint:false, tag:'E', Message:"空指令"})
      }
      if(i==0){
        this.Focus=0;//插入的这行是非active的。
        this.Code[0].active=true;
      }
      this.FreshAll();
    },
    Up:function(i){
      if(i>=1)
        this.MoveFocus((i-1)%this.Code.length);
      this.FreshAll()
    },
    Down:function(i){
      this.MoveFocus((i+1)%this.Code.length)
      this.FreshAll()
    },
    ChangeBP: function(i){
      this.Code[i].breakpoint=!this.Code[i].breakpoint;
    },
    CheckReg: function(reg){
      var reg1={
        'x0':'zero', 'x1':'ra',  'x2':'sp',  'x3':'gp',
        'x4':'tp',   'x5':'t0',  'x6':'t1',  'x7':'t2', 
        'x8':'s0',   'x9':'s1',  'x10':'a0', 'x11':'a1',
        'x12':'a2',  'x13':'a3', 'x14':'a4', 'x15':'a5',
        'x16':'a6',  'x17':'a7',  'x18':'s2', 'x19':'s3', 
        'x20':'s4',  'x21':'s5', 'x22':'s6', 'x23':'s7',
        'x24':'s8',  'x25':'s9', 'x26':'s10', 'x27':'s11',  
        'x28':'t3',  'x29':'t4', 'x30':'t5', 'x31':'t6'
      }
      var rd=reg.toLowerCase();
      if(Object.values(reg1).indexOf(rd)==-1 && Object.keys(reg1).indexOf(rd)==-1){
        return null;
      }
      if(Object.values(reg1).indexOf(rd)==-1){
        rd=reg1[rd];
      }
      return Object.values(reg1).indexOf(rd);
    },
    CheckCsr: function(reg){
      reg=reg.toLowerCase()
      var c=["mtvec","mepc","mcause","mie","mip","mtval","mscratch","mstatus"]
      var index=c.indexOf(reg)
      if(index==-1){
        return null;
      }
      return index
    },
    Compile:function(i){
      if(document.getElementById(String(i))==null){
        return true;
      } 
      this.Code[i].Message="无错误";
      this.Code[i].tag='I';
      this.Code[i].mc=0;
      this.Code[i].op='---';
      this.Code[i].rs1=0;
      this.Code[i].rs2=0;
      this.Code[i].rd=0;

      var label=null;
      var or=document.getElementById(String(i)).innerHTML;
      if(or.indexOf(":")!=-1){
        //处理标签
        or=or.split(":");
        or[0]=or[0].replace(/\s/g,'')
        if(or.length>2){
          this.Code[i].Message="标签中含有非法字符: ':'"
          this.Code[i].tag="E";
          return false;  
        }
        else if(Object.keys(this.Label).indexOf(or[0])!=-1
          &&this.Label[or[0]]!=i){
          this.Code[i].Message="已存在的标签: "+or[0];
          this.Code[i].tag="E";
          return false;  
        }
        else{
          label=or[0];
          or=or[1];
        }
      }
      or=or.split(/&nbsp|[^\w-]/).filter(Boolean);
      if(or==""){
        this.Code[i].Message="空指令";
        this.Code[i].tag="E";
        return false;
      }
      var op=or[0].toLowerCase()
      if(Object.keys(this.Inst).indexOf(op)==-1){
        this.Code[i].Message="不支持的指令"+or[0];
        this.Code[i].tag='E';
        return false;
      }
      
      switch(this.Inst[op].type){
        case('r'):
          if(or.length != 4){
            this.Code[i].Message="R型指令参数个数为3, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rd=this.CheckReg(or[1]);
          if(rd==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          var rs1=this.CheckReg(or[2]);
          if(rs1==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[2];
            this.Code[i].tag='E';
            return false;
          }            
          var rs2=this.CheckReg(or[3]);
          if(rs2==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[3];
            this.Code[i].tag='E';
            return false;
          }
          var mc=Array(32);
          for(var j=0;j<7;j++) mc[j]=this.Inst[op].fun7[j];
          for(var j=0;j<5;j++) mc[7+j]=('00000'+rs2.toString(2)).slice(-5)[j];
          for(var j=0;j<5;j++) mc[12+j]=('00000'+rs1.toString(2)).slice(-5)[j];
          for(var j=0;j<3;j++) mc[17+j]=this.Inst[op].fun3[j];
          for(var j=0;j<5;j++) mc[20+j]=('00000'+rd.toString(2)).slice(-5)[j];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j];
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rs1=rs1;
          this.Code[i].rs2=rs2;
          this.Code[i].rd=rd;
          break;
        case('i'):
          if(or.length != 4){
            this.Code[i].Message="I型指令参数个数为3, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rd=this.CheckReg(or[1]);
          if(rd==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          var rs1=this.CheckReg(or[2]);
          if(rs1==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[2];
            this.Code[i].tag='E';
            return false;
          }            
          var imm=parseInt(or[3]);
          if(isNaN(imm)){
            this.Code[i].Message="类型错误的立即数: "+or[3];
            this.Code[i].tag='E';
            return false;
          }
          if(Math.abs(imm)>0x7ff||(op[0]=='s' && Math.abs(imm)>0b01111)){
            this.Code[i].Message="立即数越界,当前为: "+String(imm);
            this.Code[i].tag='E';
            return false;
          }
          var mc=Array(32);
          for(var j=0;j<12;j++) mc[j]=('000000000000'+imm.toString(2)).slice(-12)[j];
          for(var j=0;j<5;j++) mc[12+j]=('00000'+rs1.toString(2)).slice(-5)[j];
          for(var j=0;j<3;j++) mc[17+j]=this.Inst[op].fun3[j];
          for(var j=0;j<5;j++) mc[20+j]=('00000'+rd.toString(2)).slice(-5)[j];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j]; 
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rs1=rs1;
          this.Code[i].rs2=imm;
          this.Code[i].rd=rd;
          break;
        case('l'):
          if(or.length != 4){
            this.Code[i].Message="L型指令参数个数为3, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rd=this.CheckReg(or[1]);
          if(rd==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          var rs1=this.CheckReg(or[3]);
          if(rs1==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[3];
            this.Code[i].tag='E';
            return false;
          }          
          var imm=parseInt(or[2]);
          if(isNaN(imm)){
            this.Code[i].Message="类型错误的立即数: "+or[2];
            this.Code[i].tag='E';
            return false;
          }
          if(Math.abs(imm)>0x7ff){
            this.Code[i].Message="立即数越界,当前为: "+String(imm);
            this.Code[i].tag='E';
            return false;
          }
          imm&=0xfff;
          var addr=this.Sext(imm,12)+this.Reg[rs1].value;
          addr=addr>>>2;
          if(addr>0xffff){
            this.Code[i].Message="不支持的堆地址: "+this.CalcRadix(addr,16);
            this.Code[i].tag='E';
            return false;
          }         
          var mc=Array(32);
          for(var j=0;j<12;j++) mc[j]=('000000000000'+imm.toString(2)).slice(-12)[j];
          for(var j=0;j<5;j++) mc[12+j]=('00000'+rs1.toString(2)).slice(-5)[j];
          for(var j=0;j<3;j++) mc[17+j]=this.Inst[op].fun3[j];
          for(var j=0;j<5;j++) mc[20+j]=('00000'+rd.toString(2)).slice(-5)[j];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j]; 
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rs1=rs1;
          this.Code[i].rs2=imm;
          this.Code[i].rd=rd;
          break;
        case('s'):
          if(or.length != 4){
            this.Code[i].Message="S型指令参数个数为3, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rs2=this.CheckReg(or[1]);
          if(rs2==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          var rs1=this.CheckReg(or[3]);
          if(rs1==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[3];
            this.Code[i].tag='E';
            return false;
          }          
          var imm=parseInt(or[2]);
          if(isNaN(imm)){
            this.Code[i].Message="类型错误的立即数: "+or[2];
            this.Code[i].tag='E';
            return false;
          }
          if(Math.abs(imm)>0x7ff){
            this.Code[i].Message="立即数越界,当前为: "+String(imm);
            this.Code[i].tag='E';
            return false;
          }
          imm&=0xfff;
          var addr=this.Sext(imm,7)+this.Reg[rs1].value;
          addr=addr>>>2;
          if(addr>0xffff){
            this.Code[i].Message="不支持的堆地址: "+this.CalcRadix(addr,16);
            this.Code[i].tag='E';
            return false;
          }          
          var mc=Array(32);
          for(var j=0;j<7;j++) mc[j]=('000000000000'+imm.toString(2)).slice(-12)[j];
          for(var j=0;j<5;j++) mc[7+j]=('00000'+rs2.toString(2)).slice(-5)[j];
          for(var j=0;j<5;j++) mc[12+j]=('00000'+rs1.toString(2)).slice(-5)[j];
          for(var j=0;j<3;j++) mc[17+j]=this.Inst[op].fun3[j];
          for(var j=0;j<5;j++) mc[20+j]=('000000000000'+imm.toString(2)).slice(-12)[7+j];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j]; 
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rs1=rs1;
          this.Code[i].rs2=rs2;
          this.Code[i].rd=imm;
          break;
        case('b'):
          if(or.length != 4){
            this.Code[i].Message="B型指令参数个数为3, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rs2=this.CheckReg(or[1]);
          if(rs2==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          var rs1=this.CheckReg(or[2]);
          if(rs1==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[2];
            this.Code[i].tag='E';
            return false;
          }
          if(Object.keys(this.Label).indexOf(or[3])==-1){
            this.Code[i].Message="不匹配的标签: "+or[3];
            this.Code[i].tag='E';
            return false;
          }
          var d=this.Label[or[3]];
          d=d-i;
          d=d<<2;
          if(Math.abs(d)>0xfff){
            this.Code[i].Message="立即数越界,当前为: "+this.CalcRadix(d,16);
            this.Code[i].tag='E';
            return false;            
          }
          d=d>>>0;
          var t=('0000000000000'+d.toString(2)).slice(-12);
          var mc=Array(32).fill('0');
          mc[0]=t[0];
          for(var j=0;j<6;j++) mc[1+j]=t[2+j];
          for(var j=0;j<5;j++) mc[7+j]=('00000'+rs2.toString(2)).slice(-5)[j];
          for(var j=0;j<5;j++) mc[12+j]=('00000'+rs1.toString(2)).slice(-5)[j];
          for(var j=0;j<3;j++) mc[17+j]=this.Inst[op].fun3[j];
          for(var j=0;j<4;j++) mc[20+j]=t[8+j];
          mc[24]=t[1];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j];
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rs1=rs1;
          this.Code[i].rs2=rs2;
          this.Code[i].rd=d>>0;
          break;
        case('j'):
          if(or.length != 3){
            this.Code[i].Message="J型指令参数个数为2, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rd=this.CheckReg(or[1]);
          if(rd==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          if(Object.keys(this.Label).indexOf(or[2])==-1){
            this.Code[i].Message="不匹配的标签: "+or[2];
            this.Code[i].tag='E';
            return false;
          }
          var d=this.Label[or[2]];
          d=d-i;
          d=d<<2;
          if(Math.abs(d)>0x7ffff){
            this.Code[i].Message="立即数越界,当前为: "+this.CalcRadix(d,16);
            this.Code[i].tag='E';
            return false;            
          }
          d=d>>>0;
          var t=('000000000000000000000'+d.toString(2)).slice(-21);
          var mc=Array(32).fill('0');
          mc[0]=t[0];
          for(var j=0;j<10;j++) mc[1+j]=t[10+j];
          mc[11]=t[9];
          for(var j=0;j<8;j++) mc[12+j]=t[1+j];
          for(var j=0;j<5;j++) mc[20+j]=('00000'+rd.toString(2)).slice(-5)[j];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j];
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rd=rd;
          this.Code[i].rs1=d>>0;
          break;
        case('jr'):
          if(or.length != 4){
            this.Code[i].Message="I型指令参数个数为3, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rd=this.CheckReg(or[1]);
          if(rd==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          var rs1=this.CheckReg(or[3]);
          if(rs1==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[3];
            this.Code[i].tag='E';
            return false;
          }            
          var imm=parseInt(or[2]);
          if(isNaN(imm)){
            this.Code[i].Message="类型错误的立即数: "+or[2];
            this.Code[i].tag='E';
            return false;
          }
          if(Math.abs(imm)>0x7ff){
            this.Code[i].Message="立即数越界,当前为: "+String(imm);
            this.Code[i].tag='E';
            return false;
          }
          var mc=Array(32);
          for(var j=0;j<12;j++) mc[j]=('000000000000'+imm.toString(2)).slice(-12)[j];
          for(var j=0;j<5;j++) mc[12+j]=('00000'+rs1.toString(2)).slice(-5)[j];
          for(var j=0;j<3;j++) mc[17+j]=this.Inst[op].fun3[j];
          for(var j=0;j<5;j++) mc[20+j]=('00000'+rd.toString(2)).slice(-5)[j];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j]; 
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rs1=rs1;
          this.Code[i].rs2=imm;
          this.Code[i].rd=rd;
          break;
        case('u'):
          if(or.length != 3){
            this.Code[i].Message="U型指令参数个数为2, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rd=this.CheckReg(or[1]);
          if(rd==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          var imm=parseInt(or[2]);
          if(isNaN(imm)){
            this.Code[i].Message="类型错误的立即数: "+or[2];
            this.Code[i].tag='E';
            return false;
          }
          if(Math.abs(imm)>0x7ffff){
            this.Code[i].Message="立即数越界,当前为: "+String(imm);
            this.Code[i].tag='E';
            return false;
          }
          var mc=Array(32);
          for(var j=0;j<20;j++) mc[j]=('00000000000000000000'+imm.toString(2)).slice(-20)[j];
          for(var j=0;j<5;j++) mc[20+j]=('00000'+rd.toString(2)).slice(-5)[j];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j]; 
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rs1=imm;
          this.Code[i].rd=rd;
          break;
        case('sys'):
          if(op=="mret"){
            var mc=Array(32);
            for(var j=0;j<32;j++) mc[j]=this.Inst['mret'].mc[j];
            this.Code[i].mc=parseInt(mc.join(''),2);
            this.Code[i].op=op;
            this.Code[i].rs1='';
            this.Code[i].rs2='';
            this.Code[i].rd='';  
            return;
          }
          if(or.length != 4){
            this.Code[i].Message="特权指令参数个数为3, 当前参数个数"+String(or.length-1);
            this.Code[i].tag='E';
            return false;
          }
          var rd=this.CheckReg(or[1]);
          if(rd==null){
            this.Code[i].Message="不支持的寄存器名称: "+or[1];
            this.Code[i].tag='E';
            return false;
          }
          var csr=this.CheckCsr(or[2]);
          if(csr==null){
            this.Code[i].Message="不支持的特权寄存器名称: "+or[2];
            this.Code[i].tag='E';
            return false;
          }
          var rs1; 
          if(op[op.length-1]!='i'){
            rs1=this.CheckReg(or[3]);
            if(rs1==null){
              this.Code[i].Message="不支持的寄存器名称: "+or[3];
              this.Code[i].tag='E';
              return false;
            }
          }
          else{
            var imm=parseInt(or[3]);
            if(isNaN(imm)){
              this.Code[i].Message="类型错误的立即数: "+or[3];
              this.Code[i].tag='E';
              return false;
            }
            if(Math.abs(imm)>0x7ff){
              this.Code[i].Message="立即数越界,当前为: "+String(imm);
              this.Code[i].tag='E';
              return false;
            }
            imm&=0xfff;
            rs1=imm;
          }
          var Csr=['001100000101','001101000001','001101000010','001100000100','001101000100','001101000011','001100000100','300']
          var mc=Array(32);
          for(var j=0;j<12;j++) mc[j]=Csr[csr][j];
          for(var j=0;j<5;j++) mc[12+j]=('00000'+rs1.toString(2)).slice(-5)[j];
          for(var j=0;j<3;j++) mc[17+j]=this.Inst[op].fun3[j];
          for(var j=0;j<5;j++) mc[20+j]=('00000'+rd.toString(2)).slice(-5)[j];
          for(var j=0;j<7;j++) mc[25+j]=this.Inst[op].op7[j];
          this.Code[i].mc=parseInt(mc.join(''),2);
          this.Code[i].op=op;
          this.Code[i].rs1=rs1;
          this.Code[i].rs2=csr+32;
          this.Code[i].rd=rd;  
          break;
      }
      if(label!=null){
        if(Object.keys(this.Label).indexOf(label)==-1){
          this.Label[label]=i;
        }
        else{
          var keys=Object.keys(this.Label)
          for(var j in keys){
            if(this.Label[keys[j]]==i){
              delete this.Label[keys[j]];
              this.Label[label]=i;
              break;
            }
          }
        }
      }
    },
    StackIndex:function(addr){
      //字节寻址转为内存下标
      var index=0xbffffff0-addr;
      index=index>>>2;
      return index;
    },
    StackAddr:function(index){
      var addr=index<<2;
      addr=0xbffffff0-addr;
      return addr;
    },
    TextAddr:function(index){
      var addr=index<<2;
      addr=addr+0x00010000;
      return addr;
    },
    DataAddr:function(index){
      var addr=index<<2;
      addr+=this.TextAddr(this.Code.length);
      return addr;
    },
    RunCode:function(pc){
      var code=this.Code[pc>>2]
      switch(this.Inst[code.op].type){
        case('r'):
          if(!code.rd) break;
          switch(code.op){
            case('add'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value+this.Reg[code.rs2].value;
              break;
            case('sub'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value-this.Reg[code.rs2].value;
              break;
            case('sll'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value<<this.Reg[code.rs2].value;
              break;
            case('slt'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value<this.Reg[code.rs2].value?1:0;
              break;
            case('sltu'):
              var t1=this.Reg[code.rs1].value>>>0;
              var t2=this.Reg[code.rs2].value>>>0;
              this.Reg[code.rd].value=(t1<t2)?1:0;
              break;
            case('xor'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value^this.Reg[code.rs2].value;
              break;
            case('srl'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value>>>this.Reg[code.rs2].value;
              break;
            case('sra'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value>>this.Reg[code.rs2].value;
              break;
            case('or'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value|this.Reg[code.rs2].value;
              break;
            case('and'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value&this.Reg[code.rs2].value;
              break;
          }
          break;
        case('i'):
          if(!code.rd) break;
          var imm12=this.Sext(code.rs2,12);
          switch(code.op){
            case('addi'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value+imm12;
              break;
            case('slli'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value<<code.rs2;
              break;
            case('slti'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value<code.rs2?1:0;
              break;
            case('sltiu'):
              var t1=this.Reg[code.rs1].value>>>0;
              var t2=code.rs2>>>0;
              this.Reg[code.rd].valuet1<t2?1:0;
              break;
            case('xori'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value^imm12;
              break;
            case('srli'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value>>>code.rs2;
              break;
            case('srai'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value>>code.rs2;
              break;
            case('ori'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value|imm12;
              break;
            case('andi'):
              this.Reg[code.rd].value=this.Reg[code.rs1].value&imm12;
              break;
          }
          break;
        case('l'):
          if(!code.rd) break;
          var imm12=this.Sext(code.rs2,12);
          var addr=this.Reg[code.rs1].value+imm12;
          var index=this.StackIndex(addr);
          if(index>0xffff){
            this.Code[pc>>2].tag='E';
            this.Code[pc>>2].Message="不支持的地址: "+this.CalcRadix(addr,16);
            return false;
          }
          if(this.Stack.length<index+1){
            this.Stack.push.apply(this.Stack,Array(index+1-this.Stack.length).fill(0));
          }   
          switch(code.op){
            case('lb'):              
              var t=this.Stack[index].toString(16).split('');
              var v=["",""];
              for(var i=0;i<2;i++){
                v[i]=t[addr+i];
              }
              v=parseInt(v.join(''),16);
              this.Reg[code.rd].value=this.Sext(v,8);
              break;
            case('lh'):
              if(addr%2!=0){
                //TODO:异常处理
                break;
              }
              var t=this.Stack[index].toString(16).split('');
              var v=["","","",""];
              for(var i=0;i<4;i++){
                v[i]=t[addr+i];
              }
              v=parseInt(v.join(''),16);
              this.Reg[code.rd].value=this.Sext(v,16);
              break;
            case('lw'):
              if(addr%4!=0){
                //TODO:异常处理
                break;
              }
              this.Reg[code.rd].value=this.Stack[index];
              break;
            case('lbu'):
              var t=this.Stack[index].toString(16).split('');
              var v=["",""];
              for(var i=0;i<2;i++){
                v[i]=t[addr+i];
              }
              v=parseInt(v.join(''),16);
              this.Reg[rd].value=v;
              break;
            case('lhu'):
              if(addr%2!=0){
                //TODO:异常处理
                break;
              }
              var t=this.Stack[index].toString(16).split('');
              var v=["","","",""];
              for(var i=0;i<4;i++){
                v[i]=t[addr+i];
              }
              v=parseInt(v.join(''),16);
              this.Reg[rd].value=v;
              break;
          }
          break;
        case('s'):
          var imm12=this.Sext(code.rd,12);
          var addr=this.Reg[code.rs1].value+imm12;
          var index=this.StackIndex(addr);
          if(index>0xffff){
            this.Code[pc>>2].tag='E';
            this.Code[pc>>2].Message="不支持的地址: "+this.CalcRadix(addr,16);
            return false;
          }
          if(this.Stack.length<index+1){
            this.Stack.push.apply(this.Stack,Array(1+index-this.Stack.length).fill(0));
          }   
          switch(code.op){
            case('sb'):
              var t=("00"+(this.Reg[code.rs2].value>>>0).toString(16)).slice(-2).split('');
              var v=("00000000"+(this.Stack[index]>>>0).toString(16)).slice(-8).split('');
              for(var i=0;i<2;i++){
                v[addr+i]=t[i];
              }
              v=v.join('')
              this.Stack[index]=parseInt(v,16);
              break;
            case('sh'):              
              var t=("0000"+(this.Reg[code.rs2].value>>>0).toString(16)).slice(-4).split('');
              var v=("00000000"+(this.Stack[index]>>>0).toString(16)).slice(-8).split('');
              if(addr%2!=0){
                //TODO: 异常处理
                break;
              }
              for(var i=0;i<4;i++){
                v[addr+i]=t[i];
              }
              v=v.join('');
              this.Stack[index]=parseInt(v,16);
              break;
            case('sw'):
              var t=("00000000"+(this.Reg[code.rs2].value>>>0).toString(16)).slice(-8);
              if(addr%4!=0){
                //TODO: 异常处理
                break;
              }
              this.Stack[index]=parseInt(t,16);
              break;
          }
          break;        
        case('b'):
          var dest=pc+code.rd;
          switch(code.op){
            case('beq'):
              if(code.rs1==code.rs2) this.pc=dest;
              break;
            case('bne'):
              if(code.rs1!=code.rs2) this.pc=dest;
              break;
            case('blt'):
              if(code.rs1<code.rs2) this.pc=dest;
              break;
            case('bge'):
              if(code.rs1>=code.rs2) this.pc=dest;
              break;
            case('bltu'):
              if((code.rs1>>>0)<(code.rs2>>>0)) this.pc=dest;
              break;
            case('bgeu'):
              if((code.rs1>>>0)>=(code.rs2>>>0)) this.pc=dest;
              break;
          }
          break;
        case('j'):
          this.Reg[code.rd].value=this.pc+4;
          this.pc+=code.rs1;
          break;
        case('jr'):
          if(code.rd){
            this.Reg[code.rd].value=this.pc+4;
          }
          this.pc=this.Reg[code.rs1].value + code.rs2;
          this.pc=this.pc>>1;
          this.pc=this.pc<<1;
          break;
        case('u'):
          switch(code.op){
            case('lui'):
              if(code.rd){
                this.Reg[code.rd].value=code.rs1<<12;
              }
              break;
            case('auipc'):
              if(code.rd){
                this.Reg[code.rd].value=this.pc+code.rs1<<12;
              }
              break;
          }
          break;
        case('sys'):
          if(!code.rd) break;
          switch(code.op){
            case('csrrc'):
              this.Reg[code.rd].value=this.Reg[code.rs2].value;
              this.Reg[code.rs2].value&=~this.Reg[this.rs1]
              break;
            case('csrrci'):
              this.Reg[code.rd].value=this.Reg[code.rs2].value;
              this.Reg[code.rs2].value&=~this.rs1
              break;
            case('csrrs'):
              this.Reg[code.rd].value=this.Reg[code.rs2].value;
              this.Reg[code.rs2].value|=this.Reg[this.rs1]
              break;
            case('csrrsi'):
              this.Reg[code.rd].value=this.Reg[code.rs2].value;
              this.Reg[code.rs2].value|=this.rs1
              break;              
            case('csrrw'):
              this.Reg[code.rd].value=this.Reg[code.rs2].value;
              this.Reg[code.rs2].value=this.Reg[this.rs1]
              break;
            case('csrrwi'):
              this.Reg[code.rd].value=this.Reg[code.rs2].value;
              this.Reg[code.rs2].value=this.rs1
              break;
            case('mret'):
              this.pc=this.Reg[33].value
              //todo
              break; 
          }
          break;
      }
      return true;
    },
    Step: function(){
      this.MoveFocus(null);
      for(var i in this.Code){
        if(this.Code[i].tag=="E") return;
      }
      if(this.pc==-1){
        this.pc=0;
        this.Code[this.pc>>2].run=true;
        return;
      }
      var curpc=this.pc;
      var type=this.Inst[this.Code[this.pc>>2].op].type;
      var done=this.RunCode(curpc);
      if(!done){
        //运行时错误
        this.Code[curpc>>2].run=false;
        this.pc=-1;  
        return;
      }
      this.Code[curpc>>2].run=false;
      if(type!='b'&&type!='j'&&type!='jr'){
          this.pc+=4;
      }
      if((this.pc>>2)!=this.Code.length){
        this.Code[this.pc>>2].run=true;
        return;
      }    
      this.pc=-1;
    },
    Run: function(){
      this.MoveFocus(null);
      if(this.Output!="未发现错误;<br>"){
        return;
      } 
      if(this.pc==-1){
        this.pc=0;
      }
      var n=0;
      this.DoRun=true; 
      while(this.pc!=this.Code.length){       
        if(n>0xfff){
          this.Code[this.pc].Message="运行超时";
          this.Code[this.pc].tag="E";
          this.MoveFocus(this.pc);
          this.pc=-1;
          this.DoRun=false;
          return;
        }
        n+=1;
        var curpc=this.pc;
        this.Code[curpc].run=true;
        var done=this.RunCode(this.pc);
        if(!done){
          this.Code[curpc].run=false;
          this.MoveFocus(this.pc);
          this.pc=-1;
          this.DoRun=false;
          return;
        }
        this.Code[curpc].run=false;
        if(this.pc==curpc){//处理分支
          this.pc++;
        } 
      }
      //运行结束
      this.pc-=1;
      this.MoveFocus(this.pc);
      this.DoRun=false;
    },
    Debug:function(){
      this.MoveFocus(null);
      if(this.Output!="未发现错误;<br>") return;
      if(this.pc==-1) this.pc=0;
      while(this.pc!=this.Code.length){
        var curpc=this.pc;
        this.Code[curpc].run=true;
        var done=this.RunCode(curpc);
        if(!done){
          this.Code[curpc].run=false;
          this.pc=-1;
          return;
        }
        this.Code[curpc].run=false;
        if(this.pc==curpc){
          this.pc++;
        }    
        if(this.pc!=this.Code.length&&this.Code[this.pc].breakpoint){
          this.Code[this.pc].run=true;
          return;
        }   
      }
      this.pc=-1;
      this.MoveFocus(this.Code.length-1);
    },
    Reset:function(){
      for(var j=0;j<this.Code.length;j++){
        this.Compile(j);
        this.Code[j].run=false;
        this.Code[j].breakpoint=false;
      }
      this.pc=-1;
      for(var i=0;i<this.Reg.length;i++){
        switch(this.Reg[i].name){
          case('sp'): this.Reg[i].value=0xffffffff; break;
          default: this.Reg[i].value=0; break;
        }
      }
      for(var i=0;i<this.Stack.length;i++){
        this.Stack[i]=0;
      }
      for(var i=0;i<this.Data.length;i++){
        this.Data[i]=0;
      }
    },
    ShowMachine:function(){
      this.ConsoleModel='machine';
    },
    ShowMessage:function(){
      this.ConsoleModel='message';
    },
    Edit:function(){
      this.ConsoleModel='edit'
    },
    Exp:function(){
      //处理异常

    }
  },
  computed:{
    Output:function(){
      var s='';
      switch(this.ConsoleModel){
        case('message'):
          for(var i=0; i<this.Code.length; i++){
          switch(this.Code[i].tag){
            case('E'):
              s+="行"+String(i)+"错误, "+this.Code[i].Message+";<br>"
              break;
            case('W'):
              s+="行"+String(i)+"异常, "+this.Code[i].Message+";<br>"
            }
          }
          if(s.length==0) s="未发现错误;<br>";
          break;
        case('machine'):
          for(var i=0;i<this.Code.length; i++){
            s+=this.CalcRadix(this.Code[i].mc,16)+"<br>";
          }
          break;
        case("edit"):
          for(var i=0;i<this.Code.length;i++){
            s+=this.Code[i].or+"<br>";
          }
      }
      return s;
    }
  }
}
</script>

<style>
  .outline {
    font-size: 3em;
    color: lightcoral;
    padding: 5%;
    width:100%;
    overflow: hidden
  }
  hr {
    border: 0.5px solid grey;
    padding: 0;
    margin: 0;
  }
  .hr {
    border: 0.5px solid grey;
  }
  .container {
    width: 100%;
    height: 100%;
  }  
  .title {
    font-size: 20px;
    color:gray;
  }
  .reg-container {
    width: 20%;
    height: 100%;
    float: left;
    position: fixed;
    z-index: 2;
  }
  .reg-scroller {
    overflow-y: scroll;
    white-space:nowrap;
    height:85%;
  }
  .reg-table {
    width:100%;
    border-collapse: collapse;
  }
  .reg-line{
    font-family: 'Courier New', Courier, monospace;
    font-size: 20px;
  }
  .reg-name {
    font-weight: 2px;
    font-weight: bold;
    padding-top: 10px;
  }
  .reg-value {
    padding-top: 10px;
  }
  .code-container {
    width: 50%;
    float: left;
    padding-left: 22%;
    z-index: 1;
    position: fixed;
    height: 100%;
  }  
  .code-scroller {
    overflow-y: scroll;
    white-space:nowrap;
    height:65%;
    width: 100%;
  }
  .code-table {
    width: 100%;
    border-collapse:collapse;
  }
  .code-line {
    font-family: 'Courier New', Courier, monospace;
    font-size: 20px;
    border-radius: 5px;
    padding:2px;
    width: 100%;
  }  
  .code-line.active{
    background-color: whitesmoke;
    border-left-color: whitesmoke;
    border-right-color: whitesmoke;
  }
  .code-line.run{
    background-color: lightyellow;
  }
  .code-line.breakpoint{
    background-color: lightpink;
  }
  .code-line.breakpoint-run{
    background-color: lightcoral;
  }  
  .code-linenumber {
    font-size: 20px;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    color: cadetblue;
    width: 30px;
    padding-right: 5px;
  }  
  .code-mc {
    padding-top: 1%;
    width: 50px;
  }
  .code-pre{
    width: 40px;
    padding-right:50px;
    padding-top: 1%;
    padding-left: 30px;
  }
  .code-op {
    padding-right:8px;
    color: darkred;
  }
  .code-rs {
    padding-right:5px;
  }
  .code-or {
    padding-top: 1%;
  }
  .code-out {
    border: 1.5px solid lightgrey;
    border-radius: 5px;
    margin-top: 3%;
    padding:2px;
    height: 16%;
    overflow-y: scroll;
    white-space:nowrap;
    line-height: 30px;
    font-size: 20px;
    color: gray;
  }
  .stack-container {
    width: 25%;
    height: 100%;
    float: left;
    margin-left: 73%;
    position: fixed;
    z-index: 0;
  }
  .stack-switch {
    height:85%;
  }
  .stack-scroller {
    overflow-y: scroll;
    white-space:nowrap;
    height:100%;
    overflow-x: hidden;
  }
  .stack-table {
    width:100%;
    border-collapse: collapse;
  }
  .stack-line{
    font-family: 'Courier New', Courier, monospace;
    font-size: 20px;
  }
  .stack-address {
    font-weight: 2px;
    font-weight: bold;
    margin-right: 5%;
    padding-top:2%;
  }
  .stack-value{
    padding-top:2%;
  }
  .nice-button{
    font-size: 15px;
    width: 50px;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    font-weight: lighter;
    background-color: white;
    border-color: gray;
    border: 1px solid;
    border-radius:3px;
    margin: 10px;
    padding: 5px;
  }
  .nice-button.active{
    color: white;
    background-color: rgba(128, 0, 128, 0.514);
    border-color: black;
  }
  .pc {
    font-family: 'Courier New', Courier, monospace;
    font-size: 18px;
    margin-left: 10%;
    margin-top: 2%;
    width: 12%;
  }
</style>
