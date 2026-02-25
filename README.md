<!DOCTYPE html>
<html lang="en"><head><meta charset="UTF-8"><title>Sans Fight - Upgraded</title>
<link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
<style>
body{margin:0;background:#000;color:#fff;font-family:'VT323',monospace;display:flex;justify-content:center;align-items:center;height:100vh;overflow:hidden;}
#game{width:800px;height:600px;border:4px solid #fff;box-sizing:border-box;position:relative;background:#000;overflow:hidden;}
.shake{animation:shake .3s cubic-bezier(.36,.07,.19,.97) both;}
@keyframes shake{10%,90%{transform:translate3d(-2px,0,0)}20%,80%{transform:translate3d(4px,0,0)}30%,50%,70%{transform:translate3d(-8px,0,0)}40%,60%{transform:translate3d(8px,0,0)}}
#enemy-area{position:absolute;top:20px;width:100%;height:180px;display:flex;flex-direction:column;align-items:center;pointer-events:none;}
#enemy-name{font-size:28px;margin-bottom:8px;}
#enemy-sprite{font-family:'Comic Sans MS',cursive;font-size:40px;font-weight:bold;margin-bottom:8px;animation:float 3s ease-in-out infinite;transition:transform .2s;}
.dodge{transform:translateX(-150px) !important;}
@keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(-10px)}}
#enemy-hp-container{width:300px;height:16px;border:2px solid #fff;position:relative;background:#f00;}
#enemy-hp-bar{position:absolute;height:100%;background:#0f0;width:100%;}
#enemy-hp-text{margin-top:4px;font-size:20px;}
#dialogue-box{position:absolute;bottom:150px;left:20px;right:20px;min-height:100px;border:4px solid #fff;padding:15px;font-size:28px;white-space:pre-line;background:#000;display:none;line-height:1.2;}
#sans-dialogue-box{position:absolute;top:50px;left:480px;min-height:40px;border:4px solid #fff;border-radius:10px;padding:10px;background:#fff;color:#000;font-family:'Comic Sans MS',cursive;font-size:18px;white-space:pre-line;display:none;}
#sans-dialogue-box::before{content:'';position:absolute;top:50%;left:-14px;border-width:7px;border-style:solid;border-color:transparent #fff transparent transparent;transform:translateY(-50%);}
#soul-box{position:absolute;bottom:150px;left:260px;width:280px;height:120px;border:4px solid #fff;box-sizing:border-box;overflow:hidden;display:none;background:#000;}
#soul{width:16px;height:16px;background:red;position:absolute;border-radius:2px;box-shadow:inset -2px -2px 0 rgba(0,0,0,0.3);}
.bone{position:absolute;background:#fff;border-radius:8px;}
.blaster{position:absolute;width:50px;height:60px;background:url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 50 60"><path fill="%23fff" d="M10,0 h30 v10 h10 v20 h-10 v20 h-10 v10 h-10 v-10 h-10 v-20 h-10 v-20 h10 z"/><rect x="15" y="15" width="5" height="10" fill="%23000"/><rect x="30" y="15" width="5" height="10" fill="%23000"/></svg>') no-repeat;background-size:contain;animation:flash .1s infinite;}
@keyframes flash{0%{opacity:1}50%{opacity:.8}}
.blast-beam{position:absolute;background:#fff;box-shadow:0 0 20px #fff,0 0 30px cyan;}
#ui-bar{position:absolute;bottom:0;width:100%;height:130px;display:flex;flex-direction:column;justify-content:space-between;padding:10px 20px;box-sizing:border-box;opacity:0;}
#ui-menu{display:flex;gap:40px;font-size:28px;}
.menu-item{color:#ff9900;}
.menu-item.selected{color:#ff0;}
#hp-bar{display:flex;align-items:center;gap:15px;font-size:24px;}
#hp-bar-inner{width:60px;height:24px;background:#f00;position:relative;}
#hp-fill{position:absolute;height:100%;background:#ff0;width:100%;}
#sub-menu{position:absolute;bottom:130px;width:100%;height:100px;display:none;justify-content:center;align-items:center;gap:30px;font-size:24px;}
.sub-option{color:#fff;}
.sub-option.selected{color:#ff0;}
#fight-bar-container{position:absolute;bottom:230px;left:50%;transform:translateX(-50%);width:500px;height:60px;display:none;flex-direction:column;align-items:center;color:#fff;}
#fight-bar{width:480px;height:30px;border:4px solid #fff;position:relative;background:#000;margin-bottom:10px;}
#fight-zone{position:absolute;height:100%;width:60px;background:#0f0;left:210px;}
#fight-pointer{position:absolute;height:130%;top:-15%;width:6px;background:#fff;}
#damage-text{font-size:30px;font-family:'Comic Sans MS',cursive;font-weight:bold;color:#f00;}
.miss{color:#bbb !important;}
#end-screen{position:absolute;inset:0;background:#000;display:none;justify-content:center;align-items:center;flex-direction:column;font-size:40px;text-align:center;}
</style></head><body>
<div id="game">
<div id="enemy-area"><div id="enemy-name">sans</div><div id="enemy-sprite">💀</div><div id="enemy-hp-container"><div id="enemy-hp-bar"></div></div><div id="enemy-hp-text">HP: 1 / 1</div></div>
<div id="dialogue-box"></div><div id="sans-dialogue-box"></div>
<div id="soul-box"><div id="soul"></div></div>
<div id="fight-bar-container"><div id="fight-bar"><div id="fight-zone"></div><div id="fight-pointer"></div></div><div id="damage-text"></div></div>
<div id="sub-menu"></div>
<div id="ui-bar"><div id="ui-menu"><span class="menu-item selected">FIGHT</span><span class="menu-item">ACT</span><span class="menu-item">ITEM</span><span class="menu-item">MERCY</span></div>
<div id="hp-bar"><span>NAME LV 19</span><span>HP</span><div id="hp-bar-inner"><div id="hp-fill"></div></div><span id="hp-text">92 / 92</span></div></div>
<div id="end-screen"><div id="end-text"></div><div style="font-size:24px;margin-top:20px;">(refresh to restart)</div></div>
</div>
<script>
let pHP=92,pMax=92,eHP=1,eMax=1,mItems=["FIGHT","ACT","ITEM","MERCY"],mIdx=0,mEls=document.querySelectorAll(".menu-item");
let inSoul=false,canMove=false,phase="INTRO",turns=0,pureAtk=0,canHit=false,usedBlue=false,sX=0,sY=0,sCol="red",grav=false,gVal=0.2,yVel=0,sSpd=4,keys={};
let dBox=document.getElementById("dialogue-box"),sBox=document.getElementById("sans-dialogue-box"),soulB=document.getElementById("soul-box"),soul=document.getElementById("soul");
let hpF=document.getElementById("hp-fill"),hpT=document.getElementById("hp-text"),ehpF=document.getElementById("enemy-hp-bar"),ui=document.getElementById("ui-bar"),fBox=document.getElementById("fight-bar-container"),fPt=document.getElementById("fight-pointer"),dmgT=document.getElementById("damage-text"),sSprite=document.getElementById("enemy-sprite"),game=document.getElementById("game");
let tOut=null,sOut=null,pWait=false,sWait=false,pCb=null;
const TYPE_SPD=30;
function tType(el,txt,isSans,cb,adv){
  clearTimeout(isSans?sOut:tOut);el.style.display="block";el.textContent="";let i=0;
  function stp(){
    if(i>=txt.length){if(isSans)sWait=true;else pWait=adv;if(!adv&&cb)cb();return;}
    el.textContent+=txt[i];i++;
    let d=txt[i-1]===","?150:txt[i-1]==="."?500:TYPE_SPD;
    if(isSans)sOut=setTimeout(stp,d);else tOut=setTimeout(stp,d);
  } stp();
}
function sDia(t,cb,adv){tType(sBox,t.toLowerCase(),true,cb,adv);}
function pDia(t,cb,adv){tType(dBox,t,false,cb,adv);}
function cDia(){dBox.style.display="none";sBox.style.display="none";}
function uHP(){pHP=Math.max(0,pHP);hpF.style.width=(pHP/pMax*100)+"%";hpT.textContent=pHP+" / "+pMax;if(pHP<=0)die();}
function dmg(a){if(pHP<=0)return;pHP-=a;uHP();game.classList.remove("shake");void game.offsetWidth;game.classList.add("shake");}
function die(){phase="END";document.getElementById("end-screen").style.display="flex";document.getElementById("end-text").textContent="get dunked on.";}
function rectX(a,b){return !(a.right<b.left||a.left>b.right||a.bottom<b.top||a.top>b.bottom);}
function sColSet(c){sCol=c;soul.style.background=c==="blue"?"blue":"red";grav=(c==="blue");yVel=0;}
function sLoop(){
  if(inSoul&&canMove){
    let bR=soulB.getBoundingClientRect();
    if(!grav){if(keys["ArrowUp"])sY-=sSpd;if(keys["ArrowDown"])sY+=sSpd;}
    else{if(keys["ArrowUp"]&&sY>=bR.height-20)yVel=-4;yVel+=gVal;sY+=yVel;}
    if(keys["ArrowLeft"])sX-=sSpd;if(keys["ArrowRight"])sX+=sSpd;
    sX=Math.max(0,Math.min(bR.width-16,sX));
    sY=Math.max(0,Math.min(bR.height-16,sY));
    if(sY>=bR.height-16&&grav)yVel=0;
    soul.style.left=sX+"px";soul.style.top=sY+"px";
  } requestAnimationFrame(sLoop);
}
function clrB(){document.querySelectorAll(".bone,.blaster,.blast-beam").forEach(e=>e.remove());}
function eSoul(){inSoul=true;soulB.style.display="block";let b=soulB.getBoundingClientRect();sX=b.width/2-8;sY=b.height/2-8;}
function xSoul(){inSoul=false;canMove=false;soulB.style.display="none";clrB();sColSet("red");}
function sBone(x,y,w,h){let b=document.createElement("div");b.className="bone";b.style.left=x+"px";b.style.top=y+"px";b.style.width=w+"px";b.style.height=h+"px";soulB.appendChild(b);return b;}
function sAtk(){
  phase="ATK";turns++;eSoul();canMove=true;cDia();
  if(!usedBlue){
    usedBlue=true;
    sDia("you're blue now.\nthat's my attack!",()=>{
      sColSet("blue");
      let w=soulB.getBoundingClientRect().width,b1=sBone(w,100,20,100),b2=sBone(w+150,80,20,120),bs=[b1,b2];
      let t=setInterval(()=>{
        if(!inSoul)clearInterval(t);
        bs.forEach(b=>{let x=parseFloat(b.style.left)-5;b.style.left=x+"px";if(rectX(b.getBoundingClientRect(),soul.getBoundingClientRect()))dmg(1);});
        if(parseFloat(bs[1].style.left)< -50){clearInterval(t);xSoul();phase="TURN";pDia("* sans is winking.",null,false);}
      },16);
    },true);
  } else {
    let w=soulB.getBoundingClientRect().width,bs=[];
    for(let i=0;i<6;i++){bs.push({e:sBone(w+i*80,Math.random()*80,15,50),x:w+i*80});}
    let t=setInterval(()=>{
      if(!inSoul)clearInterval(t);
      bs.forEach(b=>{b.x-=4;b.e.style.left=b.x+"px";if(rectX(b.e.getBoundingClientRect(),soul.getBoundingClientRect()))dmg(1);});
      if(bs[5].x< -20){clearInterval(t);xSoul();phase="TURN";pDia("* the scent of ketchup fills the air.",null,false);}
    },16);
  }
}
let fPos=0,fDir=1,fRun=false;
function fLoop(){
  if(fRun){
    fPos+=fDir*0.03;if(fPos<0){fPos=0;fDir=1;}if(fPos>1){fPos=1;fDir=-1;}
    fPt.style.left=(fPos*480)+"px";
  } requestAnimationFrame(fLoop);
}
function atk(){
  fRun=false;fBox.style.display="none";sSprite.classList.add("dodge");dmgT.className="miss";dmgT.textContent="MISS";
  sDia("what, you think i'm just\ngonna stand there and take it?",()=>{
    setTimeout(()=>{sSprite.classList.remove("dodge");dmgT.textContent="";sAtk();},1000);
  },true);
}
document.addEventListener("keydown",e=>{
  keys[e.key]=true;
  if(phase==="TURN"){
    if(e.key==="ArrowLeft")mIdx=Math.max(0,mIdx-1);if(e.key==="ArrowRight")mIdx=Math.min(3,mIdx+1);
    mEls.forEach((el,i)=>el.classList.toggle("selected",i===mIdx));
    if(e.key==="z"||e.key==="Enter"){
      if(mIdx===0){phase="FIGHT";cDia();fBox.style.display="flex";fRun=true;}
      else if(mIdx===3){phase="MERCY";pDia("* you spare him.",()=>{sDia("geeeeettttttt dunked on!!!",()=>{pHP=0;uHP();},true);},false);}
    }
  } else if(phase==="FIGHT"&&(e.key==="z"||e.key==="Enter")){atk();}
});
document.addEventListener("keyup",e=>keys[e.key]=false);
setTimeout(()=>{ui.style.opacity=1;phase="TURN";pDia("* you feel like you're going to have a bad time.",null,false);},1000);
requestAnimationFrame(sLoop);requestAnimationFrame(fLoop);
</script></body></html>
