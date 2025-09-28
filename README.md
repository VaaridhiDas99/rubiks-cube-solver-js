<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Rubik's Cube Solver (JS)</title>
<style>
body{font-family:system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;margin:16px;background:#f6f7fb;color:#111}
.container{display:grid;grid-template-columns:360px 1fr;gap:20px;align-items:start}
.card{background:#fff;border-radius:10px;padding:14px;box-shadow:0 6px 18px rgba(12,20,40,0.06)}
h2{margin:0 0 10px;font-size:18px}
.controls{display:flex;flex-wrap:wrap;gap:8px;margin-bottom:10px}
button{padding:8px 10px;border-radius:8px;border:1px solid #e3e7ef;background:#f8f9fc;cursor:pointer}
button.primary{background:#0b69ff;color:#fff;border-color:#0b69ff}
#cubeSvg{width:100%;height:auto}
.moves{font-family:monospace;background:#0f1724;color:#f8fafc;padding:8px;border-radius:6px;white-space:pre-wrap}
.inputRow{display:flex;gap:8px;align-items:center;margin-bottom:8px}
.legend{display:flex;gap:6px;flex-wrap:wrap;margin-top:12px}
.legend span{display:flex;align-items:center;gap:6px;padding:6px 8px;border-radius:8px;background:#fbfcff;border:1px solid #eef2ff}
.stepsList{max-height:420px;overflow:auto;padding:8px;border-radius:8px;background:#fbfdff;border:1px solid #eef2ff}
.stepItem{padding:8px;border-radius:8px;background:#fff;margin-bottom:8px;border:1px solid #eef2ff;display:flex;gap:10px;align-items:center}
.stepItem svg{width:120px;height:auto;border-radius:6px;background:#fff}
.smallBtn{padding:6px 8px;font-size:13px}
</style>
</head>
<body>
<div class="container">
<div class="card">
<h2>Rubik's Cube (Controls)</h2>
<div class="controls">
<button id="scrambleBtn" class="primary">Generate Scramble</button>
<button id="solveBtn">Solve (reverse scramble)</button>
<button id="resetBtn">Reset to Solved</button>
</div>
<div class="inputRow">
<input id="manualMove" placeholder="Enter moves e.g. R U R' F2" style="flex:1;padding:8px;border-radius:8px;border:1px solid #e9eef8">
<button id="applyMoveBtn" class="smallBtn">Apply</button>
</div>
<div style="margin-bottom:8px">
<label>Scramble length:</label>
<select id="scrambleLen">
<option>20</option><option>40</option><option selected>60</option><option>100</option>
</select>
</div>
<div style="margin-bottom:10px">
<div><strong>Current Moves History</strong></div>
<div id="history" class="moves"></div>
</div>
<div>
<div class="legend">
<span><svg width="16" height="16"><rect width="16" height="16" fill="#ff0000"></rect></svg> R</span>
<span><svg width="16" height="16"><rect width="16" height="16" fill="#00ff00"></rect></svg> G</span>
<span><svg width="16" height="16"><rect width="16" height="16" fill="#0000ff"></rect></svg> B</span>
<span><svg width="16" height="16"><rect width="16" height="16" fill="#ffff00"></rect></svg> Y</span>
<span><svg width="16" height="16"><rect width="16" height="16" fill="#ff9900"></rect></svg> O</span>
<span><svg width="16" height="16"><rect width="16" height="16" fill="#ffffff" stroke="#ccc"></rect></svg> W</span>
</div>
</div>
</div>
<div class="card">
<h2>Cube Visual & Steps</h2>
<div style="display:flex;gap:18px;align-items:flex-start">
<div style="width:320px">
<div id="cubeContainer"></div>
</div>
<div style="flex:1">
<div style="display:flex;gap:8px;align-items:center;margin-bottom:8px">
<button id="showStepsBtn" class="smallBtn">Show Steps</button>
<button id="clearStepsBtn" class="smallBtn">Clear Steps</button>
</div>
<div class="stepsList" id="stepsList"></div>
</div>
</div>
</div>
</div>
<script>
class Cube{
constructor(stateString){
if(stateString){
this.state=stateString.split('')
}else{
this.state=[]
const faces=['y','r','g','w','o','b']
for(let f=0;f<6;f++){
for(let i=0;i<9;i++)this.state.push(faces[f])
}
}
this.moveHistory=[]
}
clone(){return new Cube(this.state.join(''))}
toColorString(){return this.state.join('')}
get(i){return this.state[i]}
set(i,v){this.state[i]=v}
rotateFaceCW(base){
const s=this.state
const tmp=[s[base+0],s[base+1],s[base+2],s[base+3],s[base+4],s[base+5],s[base+6],s[base+7],s[base+8]]
s[base+0]=tmp[6];s[base+1]=tmp[3];s[base+2]=tmp[0];s[base+3]=tmp[7];s[base+4]=tmp[4];s[base+5]=tmp[1];s[base+6]=tmp[8];s[base+7]=tmp[5];s[base+8]=tmp[2]
}
applyMoveNotation(notation){
const moves=notation.trim().split(/\s+/).filter(Boolean)
for(const mv of moves)this.applySingleMove(mv)
}
applySingleMove(mv){
const baseFaces={U:0,R:9,F:18,D:27,L:36,B:45}
const face=mv[0]
let times=1
if(mv.endsWith("2"))times=2
if(mv.endsWith("'"))times=3
for(let t=0;t<times;t++)this.applyBasicMove(face)
this.moveHistory.push(mv)
}
applyBasicMove(face){
const s=this.state
if(face==='U'){
this.rotateFaceCW(0)
const cycle=[36,37,38,18,19,20,9,10,11,45,46,47]
const tmp=[s[cycle[0]],s[cycle[1]],s[cycle[2]]]
for(let i=0;i<9;i+=3)s[cycle[i]] = s[cycle[(i+3)%12]]
for(let i=1;i<9;i+=3)s[cycle[i]] = s[cycle[(i+3)%12]]
for(let i=2;i<9;i+=3)s[cycle[i]] = s[cycle[(i+3)%12]]
s[cycle[9]]=tmp[0];s[cycle[10]]=tmp[1];s[cycle[11]]=tmp[2]
}
if(face==='R'){
this.rotateFaceCW(9)
const tmp=[s[2],s[5],s[8]]
s[2]=s[18+2];s[5]=s[18+5];s[8]=s[18+8]
s[18+2]=s[27+2];s[18+5]=s[27+5];s[18+8]=s[27+8]
s[27+2]=s[45+6];s[27+5]=s[45+3];s[27+8]=s[45+0]
s[45+6]=tmp[0];s[45+3]=tmp[1];s[45+0]=tmp[2]
}
if(face==='F'){
this.rotateFaceCW(18)
const tmp=[s[6],s[7],s[8]]
s[6]=s[36+8];s[7]=s[36+5];s[8]=s[36+2]
s[36+8]=s[27+2];s[36+5]=s[27+1];s[36+2]=s[27+0]
s[27+2]=s[9+0];s[27+1]=s[9+3];s[27+0]=s[9+6]
s[9+0]=tmp[0];s[9+3]=tmp[1];s[9+6]=tmp[2]
}
if(face==='D'){
this.rotateFaceCW(27)
const cycle=[24,25,26,42,43,44,51,52,53,15,16,17]
const tmp=[s[cycle[0]],s[cycle[1]],s[cycle[2]]]
for(let i=0;i<9;i+=3)s[cycle[i]] = s[cycle[(i+3)%12]]
for(let i=1;i<9;i+=3)s[cycle[i]] = s[cycle[(i+3)%12]]
for(let i=2;i<9;i+=3)s[cycle[i]] = s[cycle[(i+3)%12]]
s[cycle[9]]=tmp[0];s[cycle[10]]=tmp[1];s[cycle[11]]=tmp[2]
}
if(face==='L'){
this.rotateFaceCW(36)
const tmp=[s[0],s[3],s[6]]
s[0]=s[45+8];s[3]=s[45+5];s[6]=s[45+2]
s[45+8]=s[27+0];s[45+5]=s[27+3];s[45+2]=s[27+6]
s[27+0]=s[18+0];s[27+3]=s[18+3];s[27+6]=s[18+6]
s[18+0]=tmp[0];s[18+3]=tmp[1];s[18+6]=tmp[2]
}
if(face==='B'){
this.rotateFaceCW(45)
const tmp=[s[0],s[1],s[2]]
s[0]=s[9+2];s[1]=s[9+5];s[2]=s[9+8]
s[9+2]=s[27+8];s[9+5]=s[27+7];s[9+8]=s[27+6]
s[27+8]=s[36+0];s[27+7]=s[36+3];s[27+6]=s[36+6]
s[36+0]=tmp[2];s[36+3]=tmp[1];s[36+6]=tmp[0]
}
}
}
function getCubeSvg(colorString,size=28,gap=4){
const map={'r':'#ff0000','g':'#00cc00','b':'#0033cc','y':'#ffeb3b','o':'#ff9900','w':'#ffffff'}
const arr=colorString.split('')
const sticker=(x,y,c)=>`<rect x="${x}" y="${y}" width="${size}" height="${size}" rx="${size*0.12}" ry="${size*0.12}" fill="${map[c]||'#000'}" stroke="#333" stroke-width="${size*0.06}" />`
const svgw=size*12+gap*10
const svgh=size*9+gap*6
let out=`<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 ${svgw} ${svgh}">`
const originX=(c)=>c*(size+gap)
const originY=(r)=>r*(size+gap)
const net=[{face:0,col:3,row:0},{face:1,col:6,row:3},{face:2,col:3,row:3},{face:3,col:3,row:6},{face:4,col:0,row:3},{face:5,col:9,row:3}]
for(const n of net){
const base= n.face*9
for(let r=0;r<3;r++){
for(let c=0;c<3;c++){
const x=originX(n.col+c)
const y=originY(n.row+r)
const color=arr[base + r*3 + c] || 'w'
out+=sticker(x,y,color)
}
}
}
out+='</svg>'
return out
}
const cube=new Cube()
const cubeContainer=document.getElementById('cubeContainer')
const historyDiv=document.getElementById('history')
const stepsList=document.getElementById('stepsList')
function render(){
cubeContainer.innerHTML=getCubeSvg(cube.toColorString())
historyDiv.textContent=cube.moveHistory.join(' ')
}
document.getElementById('applyMoveBtn').addEventListener('click',()=>{
const mv=document.getElementById('manualMove').value.trim()
if(!mv)return
try{cube.applyMoveNotation(mv);render()}catch(e){alert('Invalid move syntax')}
document.getElementById('manualMove').value=''
})
document.getElementById('resetBtn').addEventListener('click',()=>{
const newCube=new Cube()
cube.state=newCube.state.slice()
cube.moveHistory=[]
stepsList.innerHTML=''
render()
})
document.getElementById('scrambleBtn').addEventListener('click',()=>{
const len=Number(document.getElementById('scrambleLen').value)||60
const moves=['U','R','F','D','L','B']
const suffix=['',"'",'2']
const seq=[]
for(let i=0;i<len;i++){
const m=moves[Math.floor(Math.random()*moves.length)]
const s=suffix[Math.floor(Math.random()*suffix.length)]
seq.push(m+s)
}
cube.applyMoveNotation(seq.join(' '))
render()
})
document.getElementById('solveBtn').addEventListener('click',()=>{
if(cube.moveHistory.length===0){alert('No scramble/moves to reverse. Use Generate Scramble or make moves.');return}
const inv=[]
for(let i=cube.moveHistory.length-1;i>=0;i--){
const m=cube.moveHistory[i]
let base=m[0]
let tail=m.slice(1)
let invMove
if(tail==="")invMove=base+"'"
else if(tail==="2")invMove=base+"2"
else if(tail==="'")invMove=base
inv.push(invMove)
}
const snapshots=[]
for(const mv of inv){
cube.applySingleMove(mv)
snapshots.push({move:mv,state:cube.toColorString()})
}
render()
const frag=document.createDocumentFragment()
stepsList.innerHTML=''
snapshots.forEach((s,i)=>{
const div=document.createElement('div');div.className='stepItem'
div.innerHTML=`<div style="width:124px">${getCubeSvg(s.state,18,2)}</div><div><div style="font-weight:600">Step ${i+1}: ${s.move}</div><div style="font-family:monospace;margin-top:6px">${s.state}</div></div>`
stepsList.appendChild(div)
})
})
document.getElementById('showStepsBtn').addEventListener('click',()=>{
const all=stepsList.querySelectorAll('.stepItem')
if(all.length===0)alert('No steps recorded yet. Press Solve to compute steps.')
else window.scrollTo({top:document.body.scrollHeight,behavior:'smooth'})
})
document.getElementById('clearStepsBtn').addEventListener('click',()=>{stepsList.innerHTML=''})
render()
</script>
</body>
</html>
