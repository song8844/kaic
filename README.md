<div id="budgetTreemap"></div>

<style>
#budgetTreemap{
position:relative;
width:100%;
height:1400px;
background:#eef2f7;
overflow:hidden;
font-family:Arial,sans-serif;
border:1px solid #d0d7de;
box-sizing:border-box;
}

.budgetNode{
position:absolute;
border:1px solid #ffffff;
color:#111827;
padding:4px;
overflow:hidden;
box-sizing:border-box;
line-height:1.2;
}

.budgetNode .no{
font-size:11px;
font-weight:bold;
opacity:0.9;
margin-bottom:3px;
}

.budgetNode .title{
font-size:11px;
font-weight:bold;
word-break:break-word;
}

.budgetNode .amt{
font-size:10px;
margin-top:3px;
opacity:0.9;
}

.budgetNode.tiny .title,
.budgetNode.tiny .amt{
display:none;
}

.budgetNode.small .amt{
display:none;
}

.budgetNode.tiny{
padding:2px;
}

.budgetNode.tiny .no{
margin-bottom:0;
font-size:10px;
}
</style>

<script>

const rawData = [
"1. 감사원_전산운영경비(정보화)_10,300백만원",
"2. 개인정보보호위원회_개인정보 안전활용 선도기술 개발(R&D)_6,138백만원",
"3. 개인정보보호위원회_신뢰기반의 AI 개인정보 보호·활용 기술개발(R&D)_2,660백만원",
"4. 개인정보보호위원회_안전한 데이터 활용 지원_6,456백만원",
"5. 경찰청_112시스템운영(정보화)_21,372백만원",
"6. 경찰청_경찰정보화기반고도화(정보화)_78,334백만원",
"7. 경찰청_과학기술 기반 군중밀집 관리 기술 개발(R&D)_1,976백만원",
"8. 경찰청_과학수사시스템구축(정보화)_4,123백만원",
"9. 경찰청_과학적 범죄수사 고도화 기술 개발(R&D)_3,068백만원",
"10. 경찰청_불법 마약류 대응을 위한 현장기술 개발(R&D)_4,932백만원",
"11. 경찰청_사이버범죄수사단서통합분석및추론시스템개발(R&D)_6,136백만원",
"12. 경찰청_사이버수사 지원기술 개발(R&D)_8,736백만원",
"13. 경찰청_차세대지능형순찰플랫폼개발(R&D)_3,465백만원",
"14. 경찰청_치안데이터활용기술개발(R&D)_1,560백만원",
"15. 경찰청_형사·교통여성·청소년범죄수사역량강화_19,620백만원",
"16. 고용노동부_고용노동행정(정보화)_23,739백만원",
"17. 고용노동부_내일배움카드(고보)_508,979백만원",
"18. 고용노동부_내일배움카드(일반)_627,325백만원",
"19. 고용노동부_노동위원회정보화운영(정보화)_2,959백만원",
"20. 고용노동부_디지털 기반의 고용서비스 인프라 지원(정보화)_7,523백만원",
"21. 고용노동부_미래환경변화 대응 산업안전보건 연구개발(R&D)_1,554백만원",
"22. 고용노동부_사업주직업훈련지원금_249,668백만원",
"23. 고용노동부_일자리정보플랫폼 기반 AI 고용서비스 지원(정보화)_11,101백만원",
"24. 고용노동부_직업능력개발담당자양성 및 훈련매체개발_82,080백만원"
];

function parseLine(line){

const s=line.trim();

const numMatch=s.match(/^(\d+)[.)]?\s*(.*)$/);
if(!numMatch)return null;

const no=parseInt(numMatch[1],10);
const rest=numMatch[2].trim();

const lastUnderscore=rest.lastIndexOf("_");
if(lastUnderscore===-1)return null;

const left=rest.slice(0,lastUnderscore).trim();
const amountText=rest.slice(lastUnderscore+1).trim();

const firstUnderscore=left.indexOf("_");
if(firstUnderscore===-1)return null;

const agency=left.slice(0,firstUnderscore).trim();
const project=left.slice(firstUnderscore+1).trim();

const amountMatch=amountText.match(/([\d,.]+)\s*백만원/);
if(!amountMatch)return null;

const amount=parseFloat(amountMatch[1].replace(/,/g,""));

return{no,agency,project,amount,area:0};

}

function generateAgencyColors(items){

const agencies=[...new Set(items.map(i=>i.agency))];

const colors={};
const total=agencies.length;

agencies.forEach((agency,index)=>{

const hue=Math.round((index/total)*360);
colors[agency]=`hsl(${hue},65%,70%)`;

});

return colors;

}

function sumByArea(items){

let total=0;

for(let i=0;i<items.length;i++){
total+=items[i].area;
}

return total;

}

function normalizeAreas(items,w,h){

const totalArea=w*h;
const minArea=800;

let totalBudget=0;

items.forEach(i=>totalBudget+=i.amount);

items.forEach(i=>{
i.area=(i.amount/totalBudget)*totalArea;
});

}

function layoutSequential(items,x,y,w,h,horizontal,out){

if(items.length===1){

out.push({...items[0],x,y,w,h});
return;

}

const total=sumByArea(items);

let acc=0;
let splitIndex=1;

for(let i=0;i<items.length;i++){

acc+=items[i].area;

if(acc>=total/2){

splitIndex=i+1;
break;

}

}

if(splitIndex>=items.length)splitIndex=items.length-1;

const g1=items.slice(0,splitIndex);
const g2=items.slice(splitIndex);

const ratio=sumByArea(g1)/total;

if(horizontal){

const w1=w*ratio;

layoutSequential(g1,x,y,w1,h,false,out);
layoutSequential(g2,x+w1,y,w-w1,h,false,out);

}else{

const h1=h*ratio;

layoutSequential(g1,x,y,w,h1,true,out);
layoutSequential(g2,x,y+h1,w,h-h1,true,out);

}

}

function renderTreemap(){

const container=document.getElementById("budgetTreemap");

container.innerHTML="";

const items=rawData.map(parseLine).filter(Boolean);

const width=container.clientWidth||1000;
const height=container.clientHeight||1400;

normalizeAreas(items,width,height);

const agencyColors=generateAgencyColors(items);

const laidOut=[];

layoutSequential(items,0,0,width,height,true,laidOut);

laidOut.forEach(item=>{

const div=document.createElement("div");

div.className="budgetNode";

div.style.left=item.x+"px";
div.style.top=item.y+"px";
div.style.width=item.w+"px";
div.style.height=item.h+"px";

div.style.background=agencyColors[item.agency];

div.title=
item.no+"번\n"+
item.agency+"\n"+
item.project+"\n"+
item.amount.toLocaleString()+"백만원";

div.innerHTML=
'<div class="no">'+item.no+'번</div>'+
'<div class="title">'+item.agency+'<br>'+item.project+'</div>'+
'<div class="amt">'+item.amount.toLocaleString()+'백만원</div>';

container.appendChild(div);

});

}

renderTreemap();
window.onresize=renderTreemap;

</script>
