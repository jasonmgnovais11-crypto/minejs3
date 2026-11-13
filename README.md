(()=>{'use strict';
if(window.__mcStop){try{window.__mcStop()}catch(e){}}
const SEED=(window.MC_SEED??Math.random()*1e9)|0,H=80,SEA=22,IB=1000;
let RD=5;
const LS=[],on=(t,e,f,o)=>{t.addEventListener(e,f,o);LS.push([t,e,f,o])};
let raf=0;
const gc=document.createElement('canvas'),hc=document.createElement('canvas');
for(const c of[gc,hc])c.style.cssText='position:fixed;left:0;top:0;width:100vw;height:100vh;z-index:2147483647;background:#000;';
hc.style.background='transparent';
document.body.append(gc,hc);
const gl=gc.getContext('webgl2',{antialias:false,alpha:false});
if(!gl){gc.remove();hc.remove();alert('WebGL2 não suportado');return}
const hud=hc.getContext('2d');
let W=0,HH=0;
function resize(){W=hc.width=innerWidth;HH=hc.height=innerHeight;gc.width=Math.floor(W*.85);gc.height=Math.floor(HH*.85);gl.viewport(0,0,gc.width,gc.height)}
on(window,'resize',resize);resize();
window.__mcStop=()=>{cancelAnimationFrame(raf);for(const[t,e,f,o]of LS)t.removeEventListener(e,f,o);gc.remove();hc.remove();if(document.pointerLockElement)document.exitPointerLock();delete window.__mcStop};
 
/* ================= ATLAS DE TEXTURAS (32x32 tiles de 16px) ================= */
const TPR=32;
const AT=document.createElement('canvas');AT.width=AT.height=TPR*16;const ac=AT.getContext('2d');
let rs=1234567;const rnd=()=>(rs=(rs*1664525+1013904223)>>>0)/4294967296;
const TN={},FILES={};let tc=0;
function tile(name,fn,file){const i=tc++;TN[name]=i;if(file)FILES[name]=file;const ox=(i%TPR)*16,oy=(i/TPR|0)*16;
 fn((x,y,r,g,b,a=255)=>{ac.fillStyle=`rgba(${r|0},${g|0},${b|0},${a/255})`;ac.fillRect(ox+x,oy+y,1,1)});return i}
const dk=(c,d)=>[c[0]-d,c[1]-d,c[2]-d,255];
function nt(name,file,c,v,ex){return tile(name,p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++){const n=(rnd()-.5)*v;let col=[c[0]+n,c[1]+n,c[2]+n,255];if(ex){const r=ex(x,y,col);if(r)col=r}p(x,y,col[0],col[1],col[2],col[3])}},file)}
const cobP=(x,y,c)=>(y%5==0||(x+(y/5|0)*3)%6==0)?dk(c,45):null;
const brkP=(m)=>(x,y)=>(y%4==3||(x+(((y>>2)&1)*4))%8==7)?m:null;
const plank=(n,f,c)=>nt(n,f,c,12,(x,y,cc)=>y%4==3?dk(cc,30):null);
const bark=(n,f,c)=>nt(n,f,c,16,(x,y,cc)=>x%4==0?dk(cc,22):null);
const rings=(n,f,c)=>nt(n,f,c,12,(x,y,cc)=>{const d=Math.max(Math.abs(x-7.5),Math.abs(y-7.5));return d>6?dk(c,50):(d|0)%2?dk(cc,18):null});
function oreT(name,file,c,base=[125,125,125],vv=26){const sp=[];for(let i=0;i<7;i++)sp.push([2+rnd()*12,2+rnd()*12]);
 return nt(name,file,base,vv,(x,y)=>sp.some(s=>Math.hypot(s[0]-x,s[1]-y)<1.7)?[c[0]+(rnd()-.5)*30,c[1]+(rnd()-.5)*30,c[2]+(rnd()-.5)*30,255]:null)}
const metal=(n,f,c)=>nt(n,f,c,10,(x,y,cc)=>(x==0||y==0||x==15||y==15)?dk(cc,22):((x+y)%6==0?[Math.min(255,cc[0]+20),Math.min(255,cc[1]+20),Math.min(255,cc[2]+20),255]:null));
// --- blocos base
nt('white',null,[255,255,255],0);
nt('gt','grass_block_top',[200,200,200],34);
nt('gs','grass_block_side',[134,96,67],26,(x,y)=>y<3+(rnd()<.4?1:0)?[100+rnd()*20,165+rnd()*25,65,255]:null);
nt('dirt','dirt',[134,96,67],30);nt('stone','stone',[125,125,125],26);nt('cobble','cobblestone',[118,118,118],44,cobP);
nt('sand','sand',[219,207,142],14);nt('gravel','gravel',[130,125,122],56);
bark('logs','oak_log',[102,81,50]);rings('logt','oak_log_top',[160,130,80]);
nt('leaves','oak_leaves',[205,205,205],80);plank('planks','oak_planks',[170,135,80]);
nt('water','water_still',[40,90,200],20,(x,y,c)=>[c[0],c[1],c[2],150]);
nt('bedrock','bedrock',[60,60,60],70);nt('snow','snow',[245,248,250],8);
nt('snows','grass_block_snow',[134,96,67],26,(x,y)=>y<4?[240,245,250,255]:null);
nt('ice','ice',[150,200,240],20,(x,y,c)=>[c[0],c[1],c[2],200]);
nt('cacs','cactus_side',[30,120,40],25,(x,y)=>(x%5==2)?[190,225,170,255]:null);
nt('cact','cactus_top',[40,140,50],15,(x,y)=>(x==0||y==0||x==15||y==15)?[20,90,30,255]:null);
oreT('coal','coal_ore',[25,25,25]);oreT('iron','iron_ore',[215,170,140]);oreT('gold','gold_ore',[250,215,50]);oreT('dia','diamond_ore',[80,230,230]);
tile('glass',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++){if(x==0||y==0||x==15||y==15)p(x,y,200,230,240);else if((x+y)%7==0&&x<9)p(x,y,230,245,255,200);else p(x,y,0,0,0,0)}},'glass');
nt('bricks','bricks',[150,70,55],20,brkP([190,185,180,255]));
nt('sands','sandstone',[216,203,150],14,(x,y,c)=>y%8==7?dk(c,30):null);
nt('ctop','crafting_table_top',[160,120,70],14,(x,y)=>(x%5==0||y%5==0)?[90,60,30,255]:null);
nt('cside','crafting_table_side',[160,120,70],14,(x,y)=>y<4?[120,85,45,255]:(x>3&&x<12&&y>5&&y<12)?[60,60,70,255]:null);
nt('tnts','tnt_side',[200,50,40],10,(x,y)=>(y>5&&y<10)?[235,235,235,255]:null);
nt('tntt','tnt_top',[190,160,110],10,(x,y)=>(x==7||x==8||y==7||y==8)?[60,60,60,255]:null);
nt('obs','obsidian',[20,12,35],14);
nt('glow','glowstone',[240,200,110],40,(x,y,c)=>(x+y)%5==0?[255,240,170,255]:null);
nt('lava','lava_still',[230,90,20],50,()=>rnd()<.2?[255,200,50,255]:null);
// --- madeiras
const WOODS={
 oak:['Carvalho',[102,81,50],[170,135,80],205],
 spruce:['Pinheiro',[60,45,30],[115,85,50],150],
 birch:['Bétula',[215,215,205],[195,175,120],215],
 jungle:['Selva',[85,70,35],[160,115,80],190],
 acacia:['Acácia',[105,100,90],[170,90,50],210],
 dark_oak:['Carvalho escuro',[45,35,20],[65,43,20],175],
 mangrove:['Mangue',[75,35,40],[120,55,65],85],
 cherry:['Cerejeira',[95,55,55],[225,165,175],185],
 pale_oak:['Carvalho pálido',[190,185,175],[195,190,180],205],
 poplar:['Álamo',[105,100,95],[155,150,140],170],
 bamboo:['Bambu',[185,155,55],[195,175,85],110],
 crimson:['Carmesim',[105,25,45],[145,45,65],95],
 warped:['Distorcido',[25,105,105],[45,125,125],75]
};
for(const w in WOODS){const[,b,pl,lg]=WOODS[w];if(w!='oak'){bark(w+'_log',w+'_log',b);rings(w+'_logt',w+'_log_top',pl);plank(w+'_planks',w+'_planks',pl);nt(w+'_leaves',w+'_leaves',[lg,lg,lg],80)}
 nt(w+'_wood',w+'_wood',b,16);bark(w+'_stripped_log',w+'_stripped_log',pl);rings(w+'_stripped_log_top',w+'_stripped_log_top',pl);nt(w+'_stripped_wood',w+'_stripped_wood',pl,12);
}
nt('poplar_red_leaves','red_poplar_leaves',[155,55,45],55);nt('poplar_orange_leaves','orange_poplar_leaves',[205,115,45],55);nt('poplar_yellow_leaves','yellow_poplar_leaves',[205,180,55],45);
// --- pedras e terrenos
nt('smooth_stone','smooth_stone',[160,160,160],8);
nt('stonebr','stone_bricks',[125,125,125],14,(x,y,c)=>(y%8==7||(x+(y>>3)*4)%8==7)?dk(c,40):null);
nt('mcobble','mossy_cobblestone',[110,125,100],44,cobP);
nt('mstonebr','mossy_stone_bricks',[110,125,100],14,(x,y,c)=>(y%8==7||(x+(y>>3)*4)%8==7)?dk(c,40):null);
nt('granite','granite',[150,105,90],30);nt('diorite','diorite',[190,190,190],30,()=>rnd()<.1?[110,110,110,255]:null);nt('andesite','andesite',[135,135,135],30);
nt('cdirt','coarse_dirt',[120,85,60],44);
nt('podt','podzol_top',[95,70,40],30);nt('pods','podzol_side',[134,96,67],26,(x,y)=>y<4?[100,72,40,255]:null);
nt('myct','mycelium_top',[110,95,115],24);nt('mycs','mycelium_side',[134,96,67],26,(x,y)=>y<4?[115,100,120,255]:null);
nt('clay','clay',[160,165,180],10);nt('rsand','red_sand',[190,105,50],14);
nt('rsandt','red_sandstone_top',[180,95,40],10);nt('rsandb','red_sandstone_bottom',[180,95,40],10);nt('rsands','red_sandstone',[185,100,45],14,(x,y,c)=>y%8==7?dk(c,30):null);
nt('sandt','sandstone_top',[216,203,150],10);nt('sandb','sandstone_bottom',[216,203,150],10);
nt('terra','terracotta',[150,92,66],10);nt('pice','packed_ice',[140,180,235],20);
metal('coalb','coal_block',[25,25,25]);metal('ironb','iron_block',[220,220,220]);metal('goldb','gold_block',[250,215,60]);metal('diab','diamond_block',[90,230,220]);
metal('emb','emerald_block',[60,210,100]);metal('lapb','lapis_block',[40,70,170]);metal('redb','redstone_block',[190,20,10]);metal('copb','copper_block',[200,110,75]);
nt('qside','quartz_block_side',[235,230,224],8);nt('qtop','quartz_block_top',[238,233,227],6);nt('qbot','quartz_block_bottom',[232,227,221],6);
nt('nrack','netherrack',[110,40,40],40);nt('ssand','soul_sand',[85,65,50],24);
nt('nbricks','nether_bricks',[45,20,25],10,brkP([25,10,14,255]));nt('endst','end_stone',[220,220,160],20);
nt('sponge','sponge',[200,200,70],24,(x,y)=>((x*7+y*3)%11==0)?[150,150,40,255]:null);
nt('bookshelf','bookshelf',[170,135,80],12,(x,y)=>{if((y>2&&y<7)||(y>8&&y<13)){const k=[[150,40,40],[40,90,150],[60,130,60],[160,130,40]][(x>>1)%4];return[k[0],k[1],k[2],255]}return null});
nt('furnf','furnace_front',[118,118,118],30,(x,y)=>(x>3&&x<12&&y>7&&y<13)?[35,35,35,255]:(x>3&&x<12&&y>2&&y<6)?[60,60,60,255]:null);
nt('furns','furnace_side',[118,118,118],30,cobP);nt('furnt','furnace_top',[105,105,105],20);
nt('pumps','pumpkin_side',[215,125,20],16,(x,y,c)=>x%4==0?dk(c,30):null);nt('pumpt','pumpkin_top',[190,110,20],14,(x,y)=>(x>6&&x<9&&y>6&&y<9)?[90,110,30,255]:null);
nt('melons','melon_side',[120,170,40],20,(x,y,c)=>x%5<2?[80,130,30,255]:null);nt('melont','melon_top',[100,150,35],16);
nt('hays','hay_block_side',[205,170,30],24,(x,y,c)=>(x%4==0)?dk(c,40):(y>5&&y<10)?[120,80,30,255]:null);nt('hayt','hay_block_top',[200,165,35],20);
oreT('redore','redstone_ore',[210,20,20]);oreT('lapore','lapis_ore',[40,70,190]);oreT('emore','emerald_ore',[30,200,90]);oreT('copore','copper_ore',[210,120,80]);
oreT('nqore','nether_quartz_ore',[235,230,224],[110,40,40],40);
nt('deep','deepslate',[75,75,80],20,(x,y,c)=>(y%6==0)?dk(c,12):null);
nt('magma','magma',[140,40,10],30,()=>rnd()<.25?[255,150,30,255]:null);
nt('slant','sea_lantern',[180,215,205],20,(x,y)=>(x%8==0||y%8==0)?[90,140,130,255]:null);
nt('prism','prismarine',[90,160,145],30);
nt('bones','bone_block_side',[225,220,195],14,(x,y,c)=>x%5==0?dk(c,25):null);nt('bonet','bone_block_top',[225,220,195],14,(x,y,c)=>(Math.abs(x-7.5)+Math.abs(y-7.5)<4)?dk(c,25):null);
const WCOL={white:[233,236,236],orange:[240,118,19],magenta:[189,68,179],light_blue:[58,175,217],yellow:[248,198,39],lime:[112,185,25],pink:[237,141,172],gray:[62,68,71],light_gray:[142,142,134],cyan:[21,137,145],purple:[121,42,172],blue:[53,57,157],brown:[114,71,40],green:[84,109,27],red:[160,39,34],black:[20,21,25]};
const WPT={white:'branca',orange:'laranja',magenta:'magenta',light_blue:'azul-claro',yellow:'amarela',lime:'verde-limão',pink:'rosa',gray:'cinza',light_gray:'cinza-claro',cyan:'ciano',purple:'roxa',blue:'azul',brown:'marrom',green:'verde',red:'vermelha',black:'preta'};
for(const c in WCOL){nt('wool_'+c,c+'_wool',WCOL[c],18);nt('conc_'+c,c+'_concrete',WCOL[c],4)}
// plantas (cruzadas)
tile('torch',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,0,0,0,0);for(let y=6;y<16;y++){p(7,y,120,85,45);p(8,y,100,70,35)}for(let y=3;y<6;y++){p(7,y,255,210,60);p(8,y,255,170,30)}p(7,2,255,240,150)},'torch');
tile('tgrass',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,0,0,0,0);for(const x of[3,5,7,9,11,13]){const h=6+((x*7)%7);for(let y=16-h;y<16;y++)p(x,y,170+rnd()*30,170+rnd()*30,170+rnd()*30);}},'short_grass');
tile('dande',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,0,0,0,0);for(let y=7;y<16;y++)p(8,y,60,140,50);for(let y=4;y<8;y++)for(let x=6;x<=10;x++)p(x,y,250,225,40)},'dandelion');
tile('poppy',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,0,0,0,0);for(let y=7;y<16;y++)p(8,y,60,140,50);for(let y=3;y<8;y++)for(let x=6;x<=10;x++)p(x,y,205,30,30);p(8,5,30,20,20)},'poppy');
tile('dbush',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,0,0,0,0);for(let i=0;i<16;i++){p(8,15-i/1.6|0,120,85,45);p(3+(i>>1),6+(i>>2),120,85,45);p(13-(i>>1),5+(i>>2),120,85,45)}},'dead_bush');
// rostos de mobs
tile('cface',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,150+rnd()*20,150+rnd()*15,60);
 for(const[cx,cy]of[[4,6],[11,6]])for(let y=-2;y<=2;y++)for(let x=-2;x<=2;x++)if(x*x+y*y<=5)p(cx+x,cy+y,230,120,30);
 for(let x=3;x<=12;x++)p(x,11+((x%2)?1:0),50,35,15)});
tile('zface',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,90+rnd()*15,150+rnd()*15,80);
 for(const[a,b,c,d]of[[3,6,5,7],[10,6,12,7]])for(let y=b;y<=d;y++)for(let x=a;x<=c;x++)p(x,y,30,30,130);for(let x=5;x<=10;x++)p(x,11,40,60,40)});
tile('pface',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,240,160+rnd()*10,170);
 for(let y=9;y<=12;y++)for(let x=5;x<=10;x++)p(x,y,220,120,130);p(6,10,80,30,40);p(9,10,80,30,40);p(3,5,0,0,0);p(12,5,0,0,0)});
tile('face',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,235,235,235);
 for(let y=10;y<=13;y++)for(let x=4;x<=11;x++)p(x,y,255,225,205);p(3,6,0,0,0);p(12,6,0,0,0);p(4,6,0,0,0);p(11,6,0,0,0)});
tile('sface',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,205,205,198);
 for(const[a,b,c,d]of[[3,5,5,7],[10,5,12,7],[7,9,8,10]])for(let y=b;y<=d;y++)for(let x=a;x<=c;x++)p(x,y,40,40,40);for(let x=4;x<=11;x++)p(x,12,60,60,60)});
tile('spface',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,60+rnd()*15,45,40);for(const[x,y]of[[3,5],[4,5],[3,6],[4,6],[11,5],[12,5],[11,6],[12,6],[6,4],[9,4]])p(x,y,220,30,30);for(let x=6;x<=9;x++)p(x,11,20,10,10)});
{const ord=[...Array(256).keys()].sort(()=>rnd()-.5);
 for(let k=0;k<10;k++)tile('crack'+k,p=>{for(let i=0;i<(k+1)*22&&i<256;i++)p(ord[i]&15,ord[i]>>4,0,0,0,170)})}
 
/* ---------- pintores de ícones (itens) ---------- */
const rc=(p,x0,y0,x1,y1,c,v=0)=>{for(let y=y0;y<=y1;y++)for(let x=x0;x<=x1;x++){const n=(rnd()-.5)*v;p(x,y,c[0]+n,c[1]+n,c[2]+n)}};
const line=(p,x0,y0,x1,y1,c,w=1)=>{const n=Math.max(Math.abs(x1-x0),Math.abs(y1-y0),1);for(let i=0;i<=n;i++){const x=Math.round(x0+(x1-x0)*i/n),y=Math.round(y0+(y1-y0)*i/n);for(let k=0;k<w;k++)p(x+k,y,c[0],c[1],c[2])}};
const el=(p,cx,cy,rx,ry,c,v=20)=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)if(((x-cx)/rx)**2+((y-cy)/ry)**2<=1){const n=(rnd()-.5)*v;p(x,y,c[0]+n,c[1]+n,c[2]+n)}};
const sd=(c,d)=>[c[0]+d,c[1]+d,c[2]+d];
const WD=[120,85,45];
const toolP=(k,c)=>p=>{const d=sd(c,-45);
 if(k=='sword'){line(p,13,2,6,9,c,2);line(p,12,2,5,9,d,1);line(p,4,8,8,12,[90,60,30],1);line(p,5,8,9,12,[110,75,40],1);line(p,3,13,5,11,WD,1);line(p,4,13,6,11,[95,65,35],1);return}
 line(p,2,14,11,5,WD,1);line(p,3,14,12,5,[95,65,35],1);
 if(k=='pickaxe'){for(let x=4;x<=13;x++){p(x,3,...c);p(x,4,...d)}for(let y=3;y<=10;y++){p(12,y,...c);p(13,y,...d)}}
 else if(k=='axe'){for(let y=3;y<=7;y++){const a=y==3?9:y==7?10:8,b=y==3?12:y==7?11:13;for(let x=a;x<=b;x++)p(x,y,...(x==a?d:c))}}
 else if(k=='shovel')el(p,12,4,2.6,3.2,c,12);
 else if(k=='hoe'){for(let x=7;x<=12;x++){p(x,3,...c);p(x,4,...d)}p(7,5,...c);p(7,6,...d)}};
const armorP=(i,c)=>p=>{const d=sd(c,-50),l=sd(c,25);
 if(i==0){rc(p,3,4,12,9,c,18);rc(p,5,8,10,9,[0,0,0,0].slice(0,3));for(let y=7;y<=9;y++)for(let x=5;x<=10;x++)p(x,y,0,0,0,0);rc(p,3,4,12,4,l);rc(p,3,8,4,9,d);rc(p,11,8,12,9,d)}
 else if(i==1){rc(p,2,3,13,5,c,18);rc(p,4,6,11,13,c,18);rc(p,2,6,3,9,d);rc(p,12,6,13,9,d);rc(p,6,3,9,4,[0,0,0]);for(let y=3;y<=4;y++)for(let x=6;x<=9;x++)p(x,y,0,0,0,0);rc(p,7,6,8,13,d)}
 else if(i==2){rc(p,4,3,11,6,c,18);rc(p,4,7,7,13,c,18);rc(p,8,7,11,13,c,18);rc(p,4,3,11,3,l);rc(p,7,7,8,13,d);for(let y=7;y<=13;y++)p(7,y,0,0,0,0)}
 else{rc(p,3,8,6,12,c,18);rc(p,9,8,12,12,c,18);rc(p,2,12,6,13,d);rc(p,9,12,13,13,d);rc(p,3,8,6,8,l);rc(p,9,8,12,8,l)}};
const ingotP=c=>p=>{for(let y=6;y<11;y++)for(let x=3+(10-y);x<10+(10-y);x++)p(x,y,c[0]-(y-6)*10,c[1]-(y-6)*10,c[2]-(y-6)*10)};
const gemP=c=>p=>{for(let y=3;y<13;y++){const w=y<8?y-3:12-y;for(let x=8-w;x<=8+w;x++)p(x,y,c[0]+(x*7%30)-15,c[1]+(y*5%20)-10,c[2])}};
const dustP=c=>p=>{for(let i=0;i<46;i++)p(3+(rnd()*10|0),6+(rnd()*6|0),c[0]+(rnd()-.5)*30,c[1]+(rnd()-.5)*30,c[2]+(rnd()-.5)*30)};
const blobP=(c,c2)=>p=>{el(p,8,8,5.5,4.5,c);el(p,8,8,3.5,2.5,c2||sd(c,25))};
const roundP=c=>p=>{el(p,8,8,5,5,c,10);p(6,6,...sd(c,60))};
const nugP=c=>p=>el(p,8,8,2.2,2.2,c,10);
const stickP=p=>{for(let i=0;i<10;i++){p(3+i,12-i,130,95,50);p(4+i,12-i,100,70,35)}};
const coalP=c=>p=>{el(p,8,8,5,4.5,c);p(6,6,...sd(c,40));p(10,9,...sd(c,30))};
const strP=p=>{for(let i=0;i<14;i++)p(2+i,13-i+((i%4<2)?0:1),235,235,235)};
const featP=p=>{line(p,3,13,12,3,[240,240,240],2);line(p,3,13,8,8,[200,200,200],1)};
const boneP=p=>{line(p,3,13,12,4,[230,228,215],2);for(const[x,y]of[[2,12],[3,14],[11,3],[13,5]])p(x,y,230,228,215)};
const paperP=p=>rc(p,4,3,11,12,[240,240,235],10);
const bookP=p=>{rc(p,3,3,12,13,[120,60,30],12);rc(p,4,4,11,12,[235,225,200]);rc(p,3,3,4,13,[90,40,20])};
const brickP=p=>rc(p,3,6,12,10,[160,70,55],18);
const rodP=c=>p=>{line(p,3,13,12,4,c,2)};
const wheatP=p=>{for(let i=0;i<4;i++)line(p,4+i*2,14,5+i*2,3,[210,175,70]);};
const bowlP=p=>{for(let y=7;y<12;y++){const w=y-7;for(let x=3+w*.5;x<=12-w*.5;x++)p(x|0,y,120,85,50)}};
const bucketP=(c,f)=>p=>{for(let y=4;y<13;y++){const w=(y-4)*.3;for(let x=4+w|0;x<=11-w|0;x++)p(x,y,c[0]-(x%2)*10,c[1],c[2])}if(f)rc(p,5,5,10,6,f)};
const bowP=p=>{for(let i=0;i<12;i++){p(3+(i<6?i*.6:(11-i)*.6)|0,2+i,120,85,45)}line(p,4,2,4,13,[220,220,220])};
const arrowP=p=>{line(p,3,13,12,4,[130,95,50]);rc(p,11,2,13,4,[150,150,150]);line(p,2,12,4,14,[230,230,230])};
const shearsP=p=>{line(p,4,13,10,5,[200,200,200]);line(p,10,13,4,5,[200,200,200]);rc(p,3,12,5,14,[160,40,40]);rc(p,9,12,11,14,[160,40,40])};
const compP=p=>{el(p,8,8,6,6,[180,180,185]);el(p,8,8,4,4,[190,50,50]);p(8,8,20,20,20)};
const clockP=p=>{el(p,8,8,6,6,[240,200,60]);el(p,8,8,4,4,[30,30,60]);line(p,8,8,8,5,[255,255,255]);line(p,8,8,10,8,[255,255,255])};
const fsP=p=>{rc(p,3,8,9,13,[170,170,170],10);rc(p,9,4,13,9,[70,70,75],10)};
const fishRodP=p=>{line(p,3,14,12,3,WD);line(p,12,3,13,11,[230,230,230])};
const meatP=(c,c2)=>p=>{el(p,8,8,6,4.5,c);el(p,8,8,4,3,c2);for(let i=0;i<3;i++)p(11+i,3+i,235,235,225)};
const breadP=p=>{rc(p,3,6,12,10,[200,150,70],14);rc(p,3,6,12,7,[225,175,90])};
const carrotP=c=>p=>{for(let i=0;i<8;i++)for(let k=0;k<(8-i)/2+1;k++)p(4+i+(k>>1),13-i-k,c[0],c[1]+k*4,c[2]);line(p,10,4,13,2,[60,150,50]);line(p,10,4,12,5,[60,150,50])};
const cookieP=p=>{el(p,8,8,5,5,[200,140,70],12);for(const[x,y]of[[6,6],[9,7],[7,10],[10,10]])p(x,y,70,40,20)};
const melonP=p=>{for(let y=4;y<12;y++){const w=y-4;for(let x=4;x<=12-0;x++)if(x-4<=w+1)p(x,y,y>9?60:230,y>9?130:80,y>9?40:80)}};
const pieP=p=>{el(p,8,9,6,4,[210,130,40]);el(p,8,8,4,2.5,[230,170,70])};
const berryP=p=>{for(const[x,y]of[[5,6],[9,5],[7,9],[10,10],[4,11]])el(p,x,y,1.8,1.8,[190,30,50])};
const fishP=c=>p=>{el(p,7,8,5,3,c);for(let i=0;i<4;i++)p(12+(i>>1),6+i,c[0]-30,c[1]-30,c[2]-30);p(4,7,0,0,0)};
const appleP=c=>p=>{el(p,8,9,5,5,c);p(8,3,100,70,30);p(9,3,60,150,50);p(10,4,60,150,50)};
const eyeP=p=>{el(p,8,8,5,4,[110,30,40]);el(p,8,8,2,2,[220,60,60])};
// ---- registro de itens
let NI=IB;const I={},IT={};
function defItem(key,name,file,painter,o={}){tile('i_'+key,painter,file);const id=NI++;I[key]=id;IT[id]={n:name,tile:TN['i_'+key],key,...o};return id}
const TM={wooden:{n:'madeira',t:0,spd:2,c:[160,120,60],dm:0},stone:{n:'pedra',t:1,spd:4,c:[130,130,130],dm:1},iron:{n:'ferro',t:2,spd:6,c:[222,222,222],dm:2},golden:{n:'ouro',t:0,spd:12,c:[250,215,60],dm:0},diamond:{n:'diamante',t:3,spd:8,c:[80,230,230],dm:3}};
const TK=[['pickaxe','Picareta','p',2],['axe','Machado','a',3],['shovel','Pá','s',1],['hoe','Enxada','h',1],['sword','Espada','w',4]];
for(const m in TM)for(const[k,pt,code,base]of TK)defItem(m+'_'+k,pt+' de '+TM[m].n,m+'_'+k,toolP(k,TM[m].c),{tool:code,tier:TM[m].t,spd:TM[m].spd,dmg:base+TM[m].dm});
const AM={leather:{n:'couro',c:[150,90,50],p:[1,3,2,1]},chainmail:{n:'malha',c:[170,170,175],p:[2,5,4,1]},iron:{n:'ferro',c:[222,222,222],p:[2,6,5,2]},golden:{n:'ouro',c:[250,215,60],p:[2,5,3,1]},diamond:{n:'diamante',c:[80,230,230],p:[3,8,6,3]}};
const AK=[['helmet','Capacete'],['chestplate','Peitoral'],['leggings','Calças'],['boots','Botas']];
for(const m in AM)AK.forEach(([k,pt],i)=>defItem(m+'_'+k,pt+' de '+AM[m].n,m+'_'+k,armorP(i,AM[m].c),{armor:i,pts:AM[m].p[i],ac:AM[m].c.map(v=>v/255)}));
defItem('stick','Graveto','stick',stickP);defItem('coal','Carvão','coal',coalP([35,35,35]));defItem('charcoal','Carvão vegetal','charcoal',coalP([60,50,45]));
defItem('iron_ingot','Barra de ferro','iron_ingot',ingotP([215,215,215]));defItem('gold_ingot','Barra de ouro','gold_ingot',ingotP([250,215,60]));defItem('copper_ingot','Barra de cobre','copper_ingot',ingotP([200,110,75]));
defItem('iron_nugget','Pepita de ferro','iron_nugget',nugP([215,215,215]));defItem('gold_nugget','Pepita de ouro','gold_nugget',nugP([250,215,60]));
defItem('diamond','Diamante','diamond',gemP([80,230,230]));defItem('emerald','Esmeralda','emerald',gemP([40,200,90]));defItem('quartz','Quartzo do Nether','quartz',gemP([235,230,224]));
defItem('lapis','Lápis-lazúli','lapis_lazuli',blobP([40,70,190]));defItem('redstone','Redstone','redstone',dustP([200,20,20]));defItem('glowdust','Pó de glowstone','glowstone_dust',dustP([240,200,90]));
defItem('gunpowder','Pólvora','gunpowder',dustP([90,90,95]));defItem('sugar','Açúcar','sugar',dustP([240,240,240]));
defItem('string','Linha','string',strP);defItem('feather','Pena','feather',featP);defItem('leather','Couro','leather',blobP([150,90,50]));defItem('bone','Osso','bone',boneP);
defItem('flint','Sílex','flint',blobP([60,60,65]));defItem('clay_ball','Bola de argila','clay_ball',roundP([160,165,180]));defItem('brick','Tijolo','brick',brickP);
defItem('paper','Papel','paper',paperP);defItem('book','Livro','book',bookP);defItem('slime','Bola de slime','slime_ball',roundP([110,190,90]));
defItem('ender_pearl','Pérola do Ender','ender_pearl',roundP([20,110,95]));defItem('blaze_rod','Vara de blaze','blaze_rod',rodP([240,170,30]));
defItem('egg','Ovo','egg',roundP([225,205,170]));defItem('wheat','Trigo','wheat',wheatP);defItem('snowball','Bola de neve','snowball',roundP([240,245,250]));defItem('bowl','Tigela','bowl',bowlP);
defItem('bucket','Balde','bucket',bucketP([190,190,195]));defItem('water_bucket','Balde de água','water_bucket',bucketP([190,190,195],[40,90,220]));defItem('lava_bucket','Balde de lava','lava_bucket',bucketP([190,190,195],[240,100,20]));
defItem('flint_steel','Isqueiro','flint_and_steel',fsP);defItem('bow','Arco','bow',bowP);defItem('arrow','Flecha','arrow',arrowP);defItem('shears','Tesoura','shears',shearsP);
defItem('fishing_rod','Vara de pescar','fishing_rod',fishRodP);defItem('compass','Bússola','compass',compP);defItem('clock','Relógio','clock',clockP);
defItem('apple','Maçã','apple',appleP([205,30,30]),{food:4});defItem('golden_apple','Maçã dourada','golden_apple',appleP([250,210,60]),{food:4,heal:6});
defItem('bread','Pão','bread',breadP,{food:5});defItem('carrot','Cenoura','carrot',carrotP([240,130,20]),{food:3});defItem('golden_carrot','Cenoura dourada','golden_carrot',carrotP([250,210,50]),{food:6});
defItem('potato','Batata','potato',blobP([200,160,90]),{food:1});defItem('baked_potato','Batata assada','baked_potato',blobP([190,130,60],[240,220,150]),{food:5});
defItem('beef','Carne crua','beef',meatP([235,225,220],[170,50,45]),{food:3});defItem('cooked_beef','Bife','cooked_beef',meatP([130,80,40],[110,60,30]),{food:8});
defItem('porkchop','Porco cru','porkchop',meatP([255,215,215],[240,140,150]),{food:3});defItem('cooked_porkchop','Porco assado','cooked_porkchop',meatP([190,130,70],[150,95,50]),{food:8});
defItem('chicken','Frango cru','chicken',meatP([240,225,210],[225,170,160]),{food:2});defItem('cooked_chicken','Frango assado','cooked_chicken',meatP([210,150,70],[170,110,50]),{food:6});
defItem('mutton','Carneiro cru','mutton',meatP([235,215,210],[200,80,90]),{food:2});defItem('cooked_mutton','Carneiro assado','cooked_mutton',meatP([160,95,60],[130,70,40]),{food:6});
defItem('rotten','Carne podre','rotten_flesh',meatP([110,120,60],[90,100,50]),{food:4});defItem('cookie','Biscoito','cookie',cookieP,{food:2});defItem('melon_slice','Fatia de melancia','melon_slice',melonP,{food:2});
defItem('pumpkin_pie','Torta de abóbora','pumpkin_pie',pieP,{food:8});defItem('berries','Frutas silvestres','sweet_berries',berryP,{food:2});defItem('spider_eye','Olho de aranha','spider_eye',eyeP,{food:2});
defItem('cod','Bacalhau cru','cod',fishP([190,160,120]),{food:2});defItem('cooked_cod','Bacalhau assado','cooked_cod',fishP([170,120,70]),{food:5});
defItem('salmon','Salmão cru','salmon',fishP([210,100,90]),{food:2});defItem('cooked_salmon','Salmão assado','cooked_salmon',fishP([200,120,80]),{food:6});
defItem('stew','Ensopado de cogumelos','mushroom_stew',bowlP,{food:6});
const boatIcon=c=>p=>{line(p,2,9,13,9,c,2);line(p,3,10,12,10,sd(c,-35),2);line(p,4,11,11,11,sd(c,-55),1);line(p,7,2,7,8,[125,90,50],1)};
for(const w of Object.keys(WOODS).filter(w=>!['bamboo','crimson','warped'].includes(w)))defItem('boat_'+w,'Barco de '+WOODS[w][0].toLowerCase(),'boat_'+w,boatIcon(WOODS[w][2]),{boat:w});
defItem('raft_bamboo','Jangada de bambu','bamboo_raft',boatIcon([195,170,85]),{boat:'bamboo'});

 
/* ================= BLOCOS ================= */
const BL=[];
function blk(id,n,t,o={}){const R=x=>typeof x=='string'?TN[x]:x;t=Array.isArray(t)?t.map(R):[R(t),R(t),R(t)];BL[id]={n,t,s:1,hard:1,...o}}
blk(0,'Ar',0,{s:0});BL[0].air=1;
const RK=(h,need=0,o={})=>({hard:h,mat:'r',need,...o}),WDm=h=>({hard:h,mat:'w'}),DT=h=>({hard:h,mat:'d'});
blk(1,'Grama',['gt','dirt','gs'],{tint:1,...DT(.6)});blk(2,'Terra','dirt',DT(.5));blk(3,'Pedra','stone',RK(1.5,0,{drop:4}));blk(4,'Pedregulho','cobble',RK(2));
blk(5,'Areia','sand',DT(.5));blk(6,'Cascalho','gravel',DT(.6));blk(7,'Tronco de carvalho',['logt','logt','logs'],{...WDm(2),log:1});
blk(8,'Folhas de carvalho','leaves',{tint:1,leaf:1,...WDm(.2)});blk(9,'Tábuas de carvalho','planks',WDm(2));
blk(10,'Água','water',{s:0,tr:1,liq:1,hard:1e9});blk(11,'Bedrock','bedrock',{hard:1e9});
blk(12,'Grama nevada',['snow','dirt','snows'],{...DT(.6),drop:2});blk(13,'Gelo','ice',{tr:1,...RK(.5,-1),drop:0});blk(14,'Cacto',['cact','cact','cacs'],DT(.4));
blk(15,'Minério de carvão','coal',RK(3,0,{drop:I.coal}));blk(16,'Minério de ferro','iron',RK(3,1,{drop:I.iron_ingot}));
blk(17,'Minério de ouro','gold',RK(3,2,{drop:I.gold_ingot}));blk(18,'Minério de diamante','dia',RK(3,2,{drop:I.diamond}));
blk(19,'Vidro','glass',{tr:1,...DT(.3),drop:0});blk(20,'Tijolos','bricks',RK(2));blk(21,'Arenito',['sandt','sandb','sands'],RK(.8));
blk(22,'Mesa de trabalho',['ctop','planks','cside'],WDm(2.5));blk(23,'TNT',['tntt','tntt','tnts'],{hard:0});
blk(24,'Obsidiana','obs',RK(50,3));blk(25,'Pedra luminosa','glow',{...DT(.3),em:1,emit:1});blk(26,'Lava','lava',{s:0,liq:1,hard:1e9,em:1});blk(27,'Neve','snow',DT(.2));
let NB=28;const B={};
function nbk(key,n,t,o){B[key]=NB;blk(NB,n,t,o);return NB++}
const LEAFC={oak:[.45,.72,.3],birch:[.5,.68,.3],spruce:[.36,.58,.36],jungle:[.3,.78,.2],acacia:[.62,.75,.25],dark_oak:[.3,.55,.2],mangrove:[.25,.55,.35],cherry:[.95,.55,.68],pale_oak:[.78,.8,.72],poplar:[.72,.68,.58],bamboo:[.55,.72,.25],crimson:[.65,.18,.28],warped:[.1,.62,.62]};
B.oak_log=7;B.oak_planks=9;B.oak_leaves=8;
for(const w of Object.keys(WOODS))if(w!='oak'){
 const n=WOODS[w][0];
 nbk(w+'_log','Tronco de '+n.toLowerCase(),[w+'_logt',w+'_logt',w+'_log'],{...WDm(2),log:1});
 nbk(w+'_planks','Tábuas de '+n.toLowerCase(),w+'_planks',WDm(2));
 nbk(w+'_leaves','Folhas de '+n.toLowerCase(),w+'_leaves',{tint:1,tc:LEAFC[w],leaf:1,...WDm(.2)});
 const woodId=nbk(w+'_wood','Madeira de '+n.toLowerCase(),w+'_wood',WDm(2));
 const strippedId=nbk(w+'_stripped_log','Tronco descascado de '+n.toLowerCase(),[w+'_stripped_log_top',w+'_stripped_log_top',w+'_stripped_log'],{...WDm(2),log:1});
 const strippedWood=nbk(w+'_stripped_wood','Madeira descascada de '+n.toLowerCase(),w+'_stripped_wood',WDm(2));
 BL[B[w+'_log']].strip=strippedId;BL[woodId].strip=strippedWood;
}

B.oak_wood=nbk('oak_wood','Madeira de carvalho','oak_wood',WDm(2));
B.oak_stripped_log=nbk('oak_stripped_log','Tronco descascado de carvalho',['oak_stripped_log_top','oak_stripped_log_top','oak_stripped_log'],{...WDm(2),log:1});
B.oak_stripped_wood=nbk('oak_stripped_wood','Madeira descascada de carvalho','oak_stripped_wood',WDm(2));
BL[B.oak_log].strip=B.oak_stripped_log;BL[B.oak_wood].strip=B.oak_stripped_wood;
B.poplar_red_leaves=nbk('poplar_red_leaves','Folhas vermelhas de álamo','poplar_red_leaves',{leaf:1,...WDm(.2)});
B.poplar_orange_leaves=nbk('poplar_orange_leaves','Folhas laranjas de álamo','poplar_orange_leaves',{leaf:1,...WDm(.2)});
B.poplar_yellow_leaves=nbk('poplar_yellow_leaves','Folhas amarelas de álamo','poplar_yellow_leaves',{leaf:1,...WDm(.2)});
B.sapling=nbk('sapling','Muda de carvalho','sapling',{cross:1,s:0,hard:0,mat:'d',tick:1,saplingType:'oak'});
for(const w of Object.keys(WOODS).filter(w=>!['oak','bamboo','crimson','warped'].includes(w)))B[w+'_sapling']=nbk(w+'_sapling','Muda de '+WOODS[w][0].toLowerCase(),'sapling_'+w,{cross:1,s:0,hard:0,mat:'d',tick:1,saplingType:w});
B.bamboo_shoot=nbk('bamboo_shoot','Broto de bambu','sapling_bamboo',{cross:1,s:0,hard:0,mat:'d'});
nbk('smooth_stone','Pedra lisa','smooth_stone',RK(2));nbk('stonebr','Tijolos de pedra','stonebr',RK(1.5));nbk('mcobble','Pedregulho musgoso','mcobble',RK(2));nbk('mstonebr','Tijolos de pedra musgosos','mstonebr',RK(1.5));
nbk('granite','Granito','granite',RK(1.5));nbk('diorite','Diorito','diorite',RK(1.5));nbk('andesite','Andesito','andesite',RK(1.5));
nbk('cdirt','Terra grossa','cdirt',DT(.5));nbk('podzol','Podzol',['podt','dirt','pods'],{...DT(.5),drop:2});nbk('mycelium','Micélio',['myct','dirt','mycs'],{...DT(.6),drop:2});
nbk('clay','Argila','clay',{...DT(.6),drop:I.clay_ball});nbk('red_sand','Areia vermelha','rsand',DT(.5));nbk('red_sandstone','Arenito vermelho',['rsandt','rsandb','rsands'],RK(.8));
nbk('terracotta','Terracota','terra',RK(1.25));nbk('packed_ice','Gelo compactado','pice',RK(.5,-1,{drop:0}));
nbk('coal_block','Bloco de carvão','coalb',RK(5));nbk('iron_block','Bloco de ferro','ironb',RK(5,1));nbk('gold_block','Bloco de ouro','goldb',RK(3,2));nbk('diamond_block','Bloco de diamante','diab',RK(5,2));
nbk('emerald_block','Bloco de esmeralda','emb',RK(5,2));nbk('lapis_block','Bloco de lápis-lazúli','lapb',RK(3,1));nbk('redstone_block','Bloco de redstone','redb',RK(5));nbk('copper_block','Bloco de cobre','copb',RK(3,1));
nbk('quartz_block','Bloco de quartzo',['qtop','qbot','qside'],RK(.8));nbk('netherrack','Netherrack','nrack',RK(.4));nbk('soul_sand','Areia de almas','ssand',DT(.5));
nbk('nether_bricks','Tijolos do Nether','nbricks',RK(2));nbk('end_stone','Pedra do End','endst',RK(3));nbk('sponge','Esponja','sponge',DT(.6));
nt('wet_sponge','wet_sponge',[160,150,65],18);B.wet_sponge=nbk('wet_sponge','Esponja molhada','wet_sponge',DT(.6));BL[B.wet_sponge].drop=B.wet_sponge;BL[B.sponge].drop=B.sponge;
nbk('bookshelf','Estante de livros',['planks','planks','bookshelf'],{...WDm(1.5),drop:I.book});nbk('furnace','Fornalha',['furnt','furnt','furnf'],RK(3.5));
nbk('pumpkin','Abóbora',['pumpt','pumpt','pumps'],WDm(1));nbk('melon','Melancia',['melont','melont','melons'],WDm(1));nbk('hay','Fardo de feno',['hayt','hayt','hays'],DT(.5));
nbk('redstone_ore','Minério de redstone','redore',RK(3,2,{drop:I.redstone,dropn:[4,5]}));nbk('lapis_ore','Minério de lápis-lazúli','lapore',RK(3,1,{drop:I.lapis,dropn:[4,8]}));
nbk('emerald_ore','Minério de esmeralda','emore',RK(3,2,{drop:I.emerald}));nbk('copper_ore','Minério de cobre','copore',RK(3,1,{drop:I.copper_ingot,dropn:[2,3]}));
nbk('quartz_ore','Minério de quartzo','nqore',RK(3,0,{drop:I.quartz}));nbk('deepslate','Ardósia profunda','deep',RK(3,0,{drop:4}));
nbk('magma','Bloco de magma','magma',{...RK(.5),em:1});nbk('sea_lantern','Lanterna do mar','slant',{...DT(.3),em:1,emit:1,drop:I.quartz});nbk('prismarine','Prismarinho','prism',RK(1.5));
nbk('bone_block','Bloco de osso',['bonet','bonet','bones'],RK(2));
for(const c in WCOL)nbk('wool_'+c,'Lã '+WPT[c],'wool_'+c,{...DT(.8),drop:undefined});
for(const c in WCOL)nbk('conc_'+c,'Concreto '+WPT[c],'conc_'+c,RK(1.8));
nbk('torch','Tocha','torch',{cross:1,s:0,hard:0,mat:'d',em:1,emit:1});
nbk('tallgrass','Grama alta','tgrass',{cross:1,s:0,hard:0,mat:'d',tint:1,dropf:()=>Math.random()<.12?I.wheat:0});
nbk('dandelion','Dente-de-leão','dande',{cross:1,s:0,hard:0,mat:'d'});nbk('poppy','Papoula','poppy',{cross:1,s:0,hard:0,mat:'d'});nbk('deadbush','Arbusto seco','dbush',{cross:1,s:0,hard:0,mat:'d',dropf:()=>Math.random()<.4?I.stick:0});
BL[8].dropf=BL[B.birch_leaves].dropf=()=>{const r=Math.random();return r<.05?I.apple:r<.15?I.stick:0};
for(const w in WOODS)BL[B[w+'_leaves']].dropf=BL[8].dropf;
const nameOf=id=>id<IB?BL[id].n:IT[id].n;
const tileOf=id=>id<IB?BL[id].t[2]:IT[id].tile;
const ALLIDS=[];for(let i=1;i<NB;i++)if(i!=11&&BL[i])ALLIDS.push(i);for(let i=IB;i<NI;i++)ALLIDS.push(i);
const TAG={planks:[9,...Object.keys(WOODS).map(w=>B[w+'_planks'])],cobble:[4,B.mcobble],coalany:[I.coal,I.charcoal]};
const TAGN={planks:'Tábuas (qualquer)',cobble:'Pedregulho',coalany:'Carvão'};
const RECIPES=[];const rec=(o,n,ing)=>RECIPES.push([o,n,ing]);
rec(9,4,[[7,1]]);for(const w in WOODS)rec(B[w+'_planks'],4,[[B[w+'_log'],1]]);
rec(I.stick,4,[['planks',2]]);rec(B.torch,4,[[I.stick,1],['coalany',1]]);rec(22,1,[['planks',4]]);rec(B.furnace,1,[['cobble',8]]);rec(B.bookshelf,1,[['planks',6],[I.book,3]]);
const MATS={wooden:'planks',stone:'cobble',iron:I.iron_ingot,golden:I.gold_ingot,diamond:I.diamond},TC=[3,3,1,2,2],SC=[2,2,2,2,1];
for(const m in MATS)TK.forEach(([k],i)=>rec(I[m+'_'+k],1,[[MATS[m],TC[i]],[I.stick,SC[i]]]));
const AMAT={leather:I.leather,iron:I.iron_ingot,golden:I.gold_ingot,diamond:I.diamond},AC=[5,8,7,4];
for(const m in AMAT)AK.forEach(([k],i)=>rec(I[m+'_'+k],1,[[AMAT[m],AC[i]]]));
rec(B.stonebr,4,[[3,4]]);rec(21,1,[[5,4]]);rec(B.red_sandstone,1,[[B.red_sand,4]]);
rec(20,1,[[I.brick,4]]);rec(I.clay_ball,4,[[B.clay,1]]);rec(B.quartz_block,1,[[I.quartz,4]]);rec(25,1,[[I.glowdust,4]]);rec(23,1,[[I.gunpowder,5],[5,4]]);
rec(I.bread,1,[[I.wheat,3]]);rec(B.hay,1,[[I.wheat,9]]);rec(I.golden_apple,1,[[I.apple,1],[I.gold_ingot,8]]);rec(B.wool_white,1,[[I.string,4]]);
for(const[b,i]of[['iron_block','iron_ingot'],['gold_block','gold_ingot'],['diamond_block','diamond'],['emerald_block','emerald'],['lapis_block','lapis'],['redstone_block','redstone'],['coal_block','coal'],['copper_block','copper_ingot']]){rec(B[b],1,[[I[i],9]]);rec(I[i],9,[[B[b],1]])}
// Carnes, peixes e batata assada são feitos na fornalha, não na bancada.
const ARM={};for(let i=IB;i<NI;i++)if(IT[i].armor!==undefined)ARM[i]=IT[i];
 
/* ================= TEXTURA GL + SHADERS ================= */
const VS=`#version 300 es
layout(location=0) in vec3 p;layout(location=1) in vec2 uv;layout(location=2) in vec3 col;layout(location=3) in vec2 lt;
uniform mat4 M;uniform vec3 eye;out vec2 vuv;out vec3 vc;out vec2 vl;out float vd;
void main(){gl_Position=M*vec4(p,1.);vuv=uv;vc=col;vl=lt;vd=length(p-eye);}`;
const FS=`#version 300 es
precision mediump float;
in vec2 vuv;in vec3 vc;in vec2 vl;in float vd;
uniform sampler2D T;
uniform float day,fn,ff,amb,sunrise,rain;
uniform vec3 fog;
out vec4 o;

// Curva clássica aproximada, calibrada pelos níveis documentados.
float bedrockCurve(float l){
 l=clamp(l,0.0,1.0);
 return pow(l,1.18);
}
void main(){
 vec4 t=texture(T,vuv);if(t.a<.05)discard;
 float daylight=clamp(day,0.0,1.0);
 float skyLevel=bedrockCurve(vl.x);
 float skyNight=.05+.12*skyLevel;
 float skyDay=.05+.93*skyLevel;
 float sky=mix(skyNight,skyDay,daylight);
 float block=bedrockCurve(vl.y);
 vec3 nightColor=vec3(.58,.68,.94);
 vec3 dayColor=vec3(.98,.99,1.0);
 vec3 skyColor=mix(nightColor,dayColor,daylight);
 skyColor=mix(skyColor,vec3(1.0,.58,.30),clamp(sunrise,0.0,1.0)*.48);
 skyColor=mix(skyColor,vec3(.72,.76,.80),clamp(rain,0.0,1.0)*.42);
 vec3 blockColor=vec3(1.0,.72,.46);
 vec3 light=skyColor*sky+blockColor*block;
 light=max(light,vec3(amb));
 light=min(light,vec3(1.0));
 vec3 c=t.rgb*vc*light;
 float f=clamp((vd-fn)/(ff-fn),0.0,1.0);
 o=vec4(mix(c,fog,f),t.a);
}`;
function shd(t,s){const x=gl.createShader(t);gl.shaderSource(x,s);gl.compileShader(x);if(!gl.getShaderParameter(x,gl.COMPILE_STATUS))console.error(gl.getShaderInfoLog(x));return x}
const prog=gl.createProgram();gl.attachShader(prog,shd(gl.VERTEX_SHADER,VS));gl.attachShader(prog,shd(gl.FRAGMENT_SHADER,FS));gl.linkProgram(prog);gl.useProgram(prog);
const U={};for(const n of['M','eye','T','day','fn','ff','fog','amb','sunrise','rain'])U[n]=gl.getUniformLocation(prog,n);
function mkTex(unit){const t=gl.createTexture();gl.activeTexture(gl.TEXTURE0+unit);gl.bindTexture(gl.TEXTURE_2D,t);
 gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MIN_FILTER,gl.NEAREST);gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MAG_FILTER,gl.NEAREST);
 gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_WRAP_S,gl.CLAMP_TO_EDGE);gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_WRAP_T,gl.CLAMP_TO_EDGE);return t}
const tex=mkTex(0),skinTex=mkTex(1);
function uploadAtlas(){gl.activeTexture(gl.TEXTURE0);gl.bindTexture(gl.TEXTURE_2D,tex);gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,gl.RGBA,gl.UNSIGNED_BYTE,AT)}
uploadAtlas();gl.activeTexture(gl.TEXTURE0);
gl.uniform1i(U.T,0);gl.enable(gl.DEPTH_TEST);gl.blendFunc(gl.SRC_ALPHA,gl.ONE_MINUS_SRC_ALPHA);
function attribs(){const S=40;gl.enableVertexAttribArray(0);gl.vertexAttribPointer(0,3,gl.FLOAT,false,S,0);gl.enableVertexAttribArray(1);gl.vertexAttribPointer(1,2,gl.FLOAT,false,S,12);
 gl.enableVertexAttribArray(2);gl.vertexAttribPointer(2,3,gl.FLOAT,false,S,20);gl.enableVertexAttribArray(3);gl.vertexAttribPointer(3,2,gl.FLOAT,false,S,32)}
function mkMesh(v,i){if(!i.length)return null;const vao=gl.createVertexArray();gl.bindVertexArray(vao);const vb=gl.createBuffer();gl.bindBuffer(gl.ARRAY_BUFFER,vb);gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(v),gl.STATIC_DRAW);
 const ib=gl.createBuffer();gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER,ib);gl.bufferData(gl.ELEMENT_ARRAY_BUFFER,new Uint32Array(i),gl.STATIC_DRAW);attribs();return{vao,vb,ib,n:i.length}}
const freeMesh=m=>{if(m){gl.deleteVertexArray(m.vao);gl.deleteBuffer(m.vb);gl.deleteBuffer(m.ib)}};
const dyn={vao:gl.createVertexArray(),vb:gl.createBuffer(),ib:gl.createBuffer()};
gl.bindVertexArray(dyn.vao);gl.bindBuffer(gl.ARRAY_BUFFER,dyn.vb);gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER,dyn.ib);attribs();
let DV=[],DI=[],DN=0;
function dflush(mode){if(DI.length){gl.bindVertexArray(dyn.vao);gl.bindBuffer(gl.ARRAY_BUFFER,dyn.vb);gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(DV),gl.DYNAMIC_DRAW);
 gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER,dyn.ib);gl.bufferData(gl.ELEMENT_ARRAY_BUFFER,new Uint32Array(DI),gl.DYNAMIC_DRAW);gl.drawElements(mode||gl.TRIANGLES,DI.length,gl.UNSIGNED_INT,0)}DV=[];DI=[];DN=0}
function persp(fov,a,n,f){const t=1/Math.tan(fov/2);return[t/a,0,0,0,0,t,0,0,0,0,(f+n)/(n-f),-1,0,0,2*f*n/(n-f),0]}
function mul(a,b){const o=new Array(16);for(let c=0;c<4;c++)for(let r=0;r<4;r++){let s=0;for(let k=0;k<4;k++)s+=a[k*4+r]*b[c*4+k];o[c*4+r]=s}return o}
function look(e,f){let zx=-f[0],zy=-f[1],zz=-f[2];let xx=zz,xy=0,xz=-zx;let l=Math.hypot(xx,xz)||1;xx/=l;xz/=l;const yx=zy*xz-zz*xy,yy=zz*xx-zx*xz,yz=zx*xy-zy*xx;
 return[xx,yx,zx,0,xy,yy,zy,0,xz,yz,zz,0,-(xx*e[0]+xy*e[1]+xz*e[2]),-(yx*e[0]+yy*e[1]+yz*e[2]),-(zx*e[0]+zy*e[1]+zz*e[2]),1]}
 
/* ================= EXTRA: tiles, itens e blocos com formas ================= */
const NB0=NB,NI0=NI;
const clr=p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,0,0,0,0)};
const DW=Object.fromEntries(Object.entries(WOODS).map(([k,v])=>[k,v[2]]));
const WPTN=Object.fromEntries(Object.entries(WOODS).map(([k,v])=>[k,v[0].toLowerCase()]));
for(const w in DW){const c=DW[w];
 nt('dr_'+w+'_b',w+'_door_bottom',c,12,(x,y,cc)=>(x<2||x>13||y>13)?dk(cc,35):((x>3&&x<12&&y>2&&y<7)?dk(cc,22):((x>=11&&x<=12&&y>7&&y<10)?[210,210,210,255]:null)));
 nt('dr_'+w+'_t',w+'_door_top',c,12,(x,y,cc)=>(x<2||x>13||y<2)?dk(cc,35):((x>3&&x<12&&y>2&&y<9)?[160,205,230,255]:null));
 nt('tr_'+w,w+'_trapdoor',c,12,(x,y,cc)=>((x%8<2)||(y%8<2))?dk(cc,30):null)}
nt('dr_iron_b','iron_door_bottom',[210,210,215],8,(x,y,cc)=>(x<2||x>13||y>13)?dk(cc,30):((x>3&&x<12&&y>2&&y<7)?dk(cc,18):null));
nt('dr_iron_t','iron_door_top',[210,210,215],8,(x,y,cc)=>(x<2||x>13||y<2)?dk(cc,30):((x>3&&x<12&&y>2&&y<9)?[160,205,230,255]:null));
nt('tr_iron','iron_trapdoor',[205,205,210],8,(x,y,cc)=>((x%8<2)||(y%8<2))?dk(cc,30):null);
tile('ladder',p=>{clr(p);for(let y=0;y<16;y++){p(2,y,120,85,45);p(13,y,120,85,45);p(3,y,100,70,35);p(12,y,100,70,35)}for(const y of[2,6,10,14])for(let x=3;x<13;x++){p(x,y,130,95,50);p(x,y+1,110,78,40)}},'ladder');
nt('chest_s','chest_front',[160,110,50],14,(x,y,cc)=>(x==0||y==0||x==15||y==15)?dk(cc,45):((x>6&&x<9&&y>4&&y<8)?[200,200,200,255]:(y==5?dk(cc,25):null)));
nt('chest_t','chest_top',[160,110,50],14,(x,y,cc)=>(x==0||y==0||x==15||y==15)?dk(cc,45):null);
nt('chest_side','chest_side',[145,95,42],12,(x,y,cc)=>(x==0||x==15||y==0||y==15)?dk(cc,38):((y==7||y==8)?dk(cc,18):null));
tile('glass_pane',p=>{clr(p);for(let y=0;y<16;y++)for(let x=0;x<16;x++){
 if(x<2||x>13||y<2||y>13)p(x,y,205,235,245,175);
 else if(x==2||x==13||y==2||y==13)p(x,y,235,250,255,145);
}});
tile('iron_bars',p=>{clr(p);for(let y=0;y<16;y++)for(let x=0;x<16;x++){
 if(x==6||x==7||x==8||x==9||y==5||y==10)p(x,y,145,150,155,255);
 if(x==6||x==9||y==5||y==10)p(x,y,205,210,215,255);
}});
nt('furnf_on','furnace_front_on',[118,118,118],30,(x,y)=>(x>3&&x<12&&y>7&&y<13)?[240,150+((x*7+y)%40),30,255]:(x>3&&x<12&&y>2&&y<6)?[60,60,60,255]:null);
nt('bed_t','bed_top',[190,40,40],14,(x,y)=>y<4?[240,240,240,255]:null);
nt('bed_s','bed_side',[150,110,60],12,(x,y)=>y<9?[190,40,40,255]:null);
nt('farm_d','farmland',[110,75,45],16);nt('farm_w','farmland_moist',[70,45,28],16);
for(let k=0;k<8;k++)tile('wheat_'+k,p=>{clr(p);const h=2+k*2;for(const x of[2,5,8,11,14])for(let y=16-h;y<16;y++)p(x,y,k==7?200:70+k*10,k==7?170:150-k*5,k==7?70:50)},'wheat_stage'+k);
tile('sapling',p=>{clr(p);for(let y=7;y<16;y++)p(8,y,100,70,35);for(let y=2;y<9;y++)for(let x=5;x<12;x++)if(Math.hypot(x-8,y-5)<3.4)p(x,y,60+rnd()*30,150+rnd()*30,50)},'oak_sapling');
for(const w of Object.keys(WOODS).filter(w=>!['oak','bamboo','crimson','warped'].includes(w))){const c=WOODS[w][2];tile('sapling_'+w,p=>{clr(p);for(let y=7;y<16;y++)p(8,y,100,70,35);for(let y=2;y<9;y++)for(let x=5;x<12;x++)if(Math.hypot(x-8,y-5)<3.4)p(x,y,c[0],c[1],c[2])},w+'_sapling')}
nt('rs_dot','redstone_dust_dot',[220,220,220],30);
nt('lamp_off','redstone_lamp',[100,70,40],20,(x,y,c)=>(x%5==0||y%5==0)?dk(c,30):[150,110,60,255]);
nt('lamp_on','redstone_lamp_on',[255,215,120],30,(x,y,c)=>(x%5==0||y%5==0)?[200,150,70,255]:[255,230,150,255]);
tile('rtorch_on',p=>{clr(p);for(let y=6;y<16;y++){p(7,y,120,85,45);p(8,y,100,70,35)}for(let y=3;y<6;y++){p(7,y,255,40,30);p(8,y,200,20,20)}p(7,2,255,120,100)},'redstone_torch');
tile('rtorch_off',p=>{clr(p);for(let y=6;y<16;y++){p(7,y,120,85,45);p(8,y,100,70,35)}for(let y=3;y<6;y++){p(7,y,90,30,30);p(8,y,70,20,20)}},'redstone_torch_off');
nt('pis_s','piston_side',[160,120,70],12,(x,y)=>y<4?[130,130,130,255]:null);
nt('pis_t','piston_top',[160,120,70],12,(x,y)=>(x<3||x>12||y<3||y>12)?[130,130,130,255]:null);
nt('pis_b','piston_bottom',[130,130,130],14);
tile('portal',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++){const v=Math.sin(x*.7+y*.3)*.5+.5;p(x,y,90+v*70,20+v*30,170+v*60,200)}},'nether_portal');
const doorP=c=>p=>{clr(p);rc(p,4,1,11,14,c,12);rc(p,6,3,9,6,[160,205,230]);p(10,9,220,220,220)};
defItem('seeds','Sementes de trigo','wheat_seeds',dustP([120,170,60]));
defItem('saddle','Sela','saddle',blobP([120,70,40]));
defItem('ender_eye','Olho do Ender','ender_eye',roundP([40,120,100]));
for(const w in DW)defItem('door_'+w,'Porta de '+WPTN[w],w+'_door',doorP(DW[w]),{door:w});
defItem('door_iron','Porta de ferro','iron_door',doorP([210,210,215]),{door:'iron'});
 
/* ---------- formas de bloco ---------- */
const attach=(n,w,d)=>[[0,.5-w,.5-w,d,.5+w,.5+w],[1-d,.5-w,.5-w,1,.5+w,.5+w],[.5-w,0,.5-w,.5+w,d,.5+w],[.5-w,1-d,.5-w,.5+w,1,.5+w],[.5-w,.5-w,0,.5+w,.5+w,d],[.5-w,.5-w,1-d,.5+w,.5+w,1]][n];
const slabBox=mt=>mt&1?[[0,.5,0,1,1,1]]:[[0,0,0,1,.5,1]];
const stairBox=mt=>{const f=mt&3,up=mt&4,y0=up?0:.5,y1=up?.5:1;return[up?[0,.5,0,1,1,1]:[0,0,0,1,.5,1],[[.5,y0,0,1,y1,1],[0,y0,0,.5,y1,1],[0,y0,.5,1,y1,1],[0,y0,0,1,y1,.5]][f]]};
const doorBox=mt=>{const f=mt&3,ff=(mt&4)?(f+1)&3:f;return[[[0,0,0,.1875,1,1],[.8125,0,0,1,1,1],[0,0,0,1,1,.1875],[0,0,.8125,1,1,1]][ff]]};
const trapBox=mt=>{const f=mt&3;if(mt&4)return[[[.8125,0,0,1,1,1],[0,0,0,.1875,1,1],[0,0,.8125,1,1,1],[0,0,0,1,1,.1875]][f]];return mt&8?[[0,.8125,0,1,1,1]]:[[0,0,0,1,.1875,1]]};
const ladBox=mt=>[[[0,0,0,.125,1,1]],[[.875,0,0,1,1,1]],[[0,0,0,1,1,.125]],[[0,0,0,1,1,.125]],[[0,0,0,1,1,.125]],[[0,0,.875,1,1,1]]][mt&7].slice(0,1).map(b=>b);
const LADB=[[0,0,0,.125,1,1],[.875,0,0,1,1,1],0,0,[0,0,0,1,1,.125],[0,0,.875,1,1,1]];
const ladBox2=mt=>[LADB[mt&7]||LADB[4]];
const fenceBox=(post,arm)=>(mt,x,y,z)=>{const r=[post],cn=(dx,dz)=>{const b=getBlock(x+dx,y,z+dz);if(!b)return false;const q=BL[b];return q.fence||(q.s&&!q.box&&!q.tr&&!q.cross)};
 if(cn(1,0))r.push(arm[0]);if(cn(-1,0))r.push(arm[1]);if(cn(0,1))r.push(arm[2]);if(cn(0,-1))r.push(arm[3]);return r};
const FENCE=fenceBox([.375,0,.375,.625,1,.625],[[.625,.375,.4375,1,.9375,.5625],[0,.375,.4375,.375,.9375,.5625],[.4375,.375,.625,.5625,.9375,1],[.4375,.375,0,.5625,.9375,.375]]);
const WALL=fenceBox([.25,0,.25,.75,1,.75],[[.75,0,.3125,1,.8125,.6875],[0,0,.3125,.25,.8125,.6875],[.3125,0,.75,.6875,.8125,1],[.3125,0,0,.6875,.8125,.25]]);
const paneBox=(mt,x,y,z)=>{const r=[[.4375,0,.4375,.5625,1,.5625]],cn=(dx,dz)=>{
 const id=getBlock(x+dx,y,z+dz);if(!id)return false;const q=BL[id];
 return q.glassPane||q.ironBars||(q.s&&!q.tr&&!q.box&&!q.cross)};
 if(cn(1,0))r.push([.5,0,.4375,1,1,.5625]);
 if(cn(-1,0))r.push([0,0,.4375,.5,1,.5625]);
 if(cn(0,1))r.push([.4375,0,.5,.5625,1,1]);
 if(cn(0,-1))r.push([.4375,0,0,.5625,1,.5]);
 return r};
const wireBox=(mt,x,y,z)=>{const r=[[.3125,0,.3125,.6875,.03,.6875]],cn=(dx,dz)=>{const b=getBlock(x+dx,y,z+dz);return b&&BL[b].rs};
 if(cn(1,0))r.push([.6875,0,.3125,1,.03,.6875]);if(cn(-1,0))r.push([0,0,.3125,.3125,.03,.6875]);if(cn(0,1))r.push([.3125,0,.6875,.6875,.03,1]);if(cn(0,-1))r.push([.3125,0,0,.6875,.03,.3125]);return r};
const leverBox=mt=>{const n=mt&7;return[[...attach(n,.25,.125),TN.cobble],[...attach(n,.0625,.5),TN.planks]]};
const buttonBox=(mt,tex=TN.stone)=>[[...attach(mt&7,.1875,(mt&8)?.0625:.125),tex]];
const plateBox=(mt,x,y,z,tex=TN.stone)=>[[.0625,0,.0625,.9375,(mt&8)?.03:.0625,.9375,tex]];
const headBox=n=>[[.75,0,0,1,1,1],[0,0,0,.25,1,1],[0,.75,0,1,1,1],[0,0,0,1,.25,1],[0,0,.75,1,1,1],[0,0,0,1,1,.25]][n];
plank('bamboo_mosaic','bamboo_mosaic',[190,165,85]);
B.bamboo_mosaic=nbk('bamboo_mosaic','Mosaico de bambu','bamboo_mosaic',WDm(2));
const SLABM=[[4,'pedregulho'],[3,'pedra'],[B.smooth_stone,'pedra lisa'],[20,'tijolos'],[B.stonebr,'tijolos de pedra'],[21,'arenito'],...Object.keys(WOODS).map(w=>[B[w+'_planks'],WOODS[w][0].toLowerCase()]),[B.bamboo_mosaic,'mosaico de bambu'],[B.quartz_block,'quartzo'],[B.nether_bricks,'tijolos do Nether'],[B.red_sandstone,'arenito vermelho'],[B.mcobble,'pedregulho musgoso']];
const SLAB={},STAIR={};
for(const[m,n]of SLABM){const b=BL[m],o={hard:b.hard,mat:b.mat,need:b.need};
 SLAB[m]=nbk('slab_'+m,'Laje de '+n,b.t.slice(),{...o,box:slabBox,slab:m});STAIR[m]=nbk('stair_'+m,'Escada de '+n,b.t.slice(),{...o,box:stairBox,stairs:1})}
for(const w of Object.keys(WOODS)){const m=B[w+'_planks'],n=WOODS[w][0].toLowerCase();nbk('fence_'+w,'Cerca de '+n,BL[m].t.slice(),{hard:2,mat:'w',box:FENCE,fence:1,tall:1})}
nbk('wall_cobble','Muro de pedregulho',BL[4].t.slice(),{hard:2,mat:'r',need:0,box:WALL,fence:1,tall:1});
nbk('wall_mossy','Muro de pedregulho musgoso',BL[B.mcobble].t.slice(),{hard:2,mat:'r',need:0,box:WALL,fence:1,tall:1});
for(const w of[...Object.keys(DW),'iron']){const tb=TN['dr_'+w+'_b'],tt=TN['dr_'+w+'_t'],ir=w=='iron';
 nbk('door_'+w,'Porta ('+w+')',[tb,tb,tb],{box:doorBox,door:1,iron:ir,tmeta:mt=>mt&8?tt:tb,hard:ir?5:3,mat:ir?'r':'w',need:ir?0:undefined,rs:'use',hide:1,dropm:mt=>(mt&8)?[]:[[I['door_'+w],1]]});
 const tr=TN['tr_'+w];nbk('trap_'+w,'Alçapão ('+w+')',[tr,tr,tr],{box:trapBox,trap:1,iron:ir,hard:ir?5:3,mat:ir?'r':'w',need:ir?0:undefined,rs:'use'})}
// Portões, botões, placas de pressão e placas decorativas para cada madeira.
const gateBox=mt=>{const open=!!(mt&4),axis=mt&1;return open?[[.0,0,.375,.125,1,.625],[.875,0,.375,1,1,.625],[.125,.375,.375,.875,.5,.625]]:axis?[[.375,0,0,.625,1,1],[0,.375,.375,1,.5,.625]]:[[0,0,.375,1,1,.625],[.375,.375,0,.625,.5,1]]};
const signBox=mt=>{const post=[.4375,0,.4375,.5625,.48,.5625];const board=(mt&3)<2?[.4375,.36,.04,.5625,.94,.96]:[.04,.36,.4375,.96,.94,.5625];return[post,board]};
for(const w of Object.keys(WOODS)){
 const nm=WOODS[w][0].toLowerCase(),pl=B[w+'_planks'],tex=BL[pl].t[2];
 nbk('gate_'+w,'Portão de '+nm,tex,{box:gateBox,gate:1,hard:2,mat:'w',fence:1});
 nbk('button_'+w,'Botão de '+nm,tex,{box:(mt)=>buttonBox(mt,tex),nocol:1,s:0,rs:'button',hard:.5,mat:'w'});
 nbk('plate_'+w,'Placa de pressão de '+nm,tex,{box:(mt,x,y,z)=>plateBox(mt,x,y,z,tex),nocol:1,s:0,rs:'plate',hard:.5,mat:'w'});
 nbk('sign_'+w,'Placa de '+nm,[tex,tex,tex],{box:signBox,nocol:1,hard:1,mat:'w',sign:1});
 nbk('hanging_sign_'+w,'Placa suspensa de '+nm,tex,{box:[[.25,.45,.25,.75,.95,.75]],nocol:1,hard:1,mat:'w',sign:1});
}
nbk('ladder','Escada de mão','ladder',{box:ladBox2,nocol:1,climb:1,s:0,hard:.4,mat:'w'});
const chestBox=(mt,x,y,z)=>{const f=mt&3,alongZ=f<2;let x0=.0625,x1=.9375,z0=.0625,z1=.9375;
 if(alongZ){if(getBlock(x,y,z-1)==B.chest&&getMeta(x,y,z-1)==mt)z0=0;if(getBlock(x,y,z+1)==B.chest&&getMeta(x,y,z+1)==mt)z1=1}
 else{if(getBlock(x-1,y,z)==B.chest&&getMeta(x-1,y,z)==mt)x0=0;if(getBlock(x+1,y,z)==B.chest&&getMeta(x+1,y,z)==mt)x1=1}
 return [[x0,0,z0,x1,.75,z1],[x0,.75,z0,x1,.875,z1]]};
const chestFace=(mt,f)=>{const front=[0,1,4,5][mt&3];return f==2?TN.chest_t:f==front?TN.chest_s:TN.chest_side};
nbk('chest','Baú',['chest_t','chest_t','chest_s'],{box:chestBox,faceTex:chestFace,cont:'chest',hard:2.5,mat:'w'});
nbk('glass_pane','Painel de vidro','glass_pane',{box:paneBox,glassPane:1,tr:1,hard:.3,mat:'d'});
nbk('iron_bars','Barras de ferro','iron_bars',{box:paneBox,ironBars:1,hard:5,mat:'r',need:1});
nbk('furnace_on','Fornalha (acesa)',['furnt','furnt','furnf_on'],{hard:3.5,mat:'r',need:0,cont:'furnace',em:1,emit:1,drop:B.furnace,hide:1});
BL[B.furnace].cont='furnace';
nbk('bed','Cama',['bed_t','planks','bed_s'],{box:[[0,0,0,1,.5625,1]],bed:1,hard:.2,mat:'w'});
nbk('farmland','Terra arada',['farm_d','dirt','dirt'],{box:[[0,0,0,1,.9375,1]],hard:.6,mat:'d',drop:2,ttop:mt=>mt&1?TN.farm_w:TN.farm_d,tick:1,hide:1});
nbk('wheat','Trigo','wheat_0',{cross:1,s:0,hard:0,mat:'d',tmeta:mt=>TN['wheat_'+(mt&7)],tick:1,hide:1,dropm:mt=>mt>=7?[[I.wheat,1],[I.seeds,1+(Math.random()*3|0)]]:[[I.seeds,1]]});
nbk('rs_wire','Pó de redstone','rs_dot',{box:wireBox,nocol:1,s:0,rs:'wire',hard:0,mat:'d',tf:mt=>[.3+((mt&15)/15)*.7,.03,.03],dropm:()=>[[I.redstone,1]]});
nbk('lever','Alavanca','cobble',{box:leverBox,nocol:1,s:0,rs:'lever',hard:.5,mat:'r',tf:mt=>(mt&8)?[1.3,1.05,1.05]:[1,1,1]});
nbk('button','Botão de pedra','stone',{box:buttonBox,nocol:1,s:0,rs:'button',hard:.5,mat:'r'});
nbk('plate','Placa de pressão','stone',{box:plateBox,nocol:1,s:0,rs:'plate',hard:.5,mat:'r'});
nbk('rtorch','Tocha de redstone','rtorch_on',{cross:1,s:0,hard:0,mat:'d',rs:'torch_on',em:1,emit:1});
nbk('rtorch_off','Tocha de redstone (apagada)','rtorch_off',{cross:1,s:0,hard:0,mat:'d',rs:'torch_off',drop:B.rtorch,hide:1});
nbk('lamp','Lâmpada de redstone','lamp_off',{hard:.3,mat:'d',rs:'lamp'});
nbk('lamp_on','Lâmpada de redstone (acesa)','lamp_on',{hard:.3,mat:'d',rs:'lamp',em:1,emit:1,drop:B.lamp,hide:1});
nbk('piston','Pistão',['pis_t','pis_b','pis_s'],{hard:1.5,mat:'r',rs:'piston'});
nbk('pistonhead','Cabeça de pistão','pis_t',{box:mt=>[headBox(mt&7)],hard:1.5,mat:'r',drop:0,hide:1});
nbk('portal','Portal do Nether','portal',{box:mt=>mt?[[.375,0,0,.625,1,1]]:[[0,0,.375,1,1,.625]],s:0,tr:1,nocol:1,em:1,hard:0,drop:0,hide:1});
BL[B.redstone_block].rs='block';BL[23].rs='use';BL[22].craft=1;
BL[5].fall=BL[6].fall=BL[B.red_sand].fall=1;
for(const i of[15,16,17,18,B.redstone_ore,B.lapis_ore,B.emerald_ore,B.copper_ore,B.quartz_ore])BL[i].oreXp=1;
BL[8].dropf=()=>{const r=Math.random();return r<.05?I.apple:r<.1?B.sapling:r<.2?I.stick:0};
for(const w of Object.keys(WOODS)){const sap=B[w+'_sapling'];if(sap)BL[B[w+'_leaves']].dropf=()=>Math.random()<.1?sap:(Math.random()<.1?I.stick:0)}
for(const id of[B.poplar_red_leaves,B.poplar_orange_leaves,B.poplar_yellow_leaves])BL[id].dropf=()=>Math.random()<.1?B.poplar_sapling:0;
for(let i=NB0;i<NB;i++)if(!BL[i].hide)ALLIDS.push(i);
for(let i=NI0;i<NI;i++)ALLIDS.push(i);
// receitas extras
for(const[m]of SLABM){rec(SLAB[m],6,[[m,3]]);rec(STAIR[m],4,[[m,6]])}
for(const w of Object.keys(WOODS)){const m=B[w+'_planks'];rec(B['fence_'+w],3,[[m,4],[I.stick,2]])}
for(const w of Object.keys(WOODS)){const m=B[w+'_planks'];rec(B['gate_'+w],1,[[I.stick,4],[m,2]]);rec(B['button_'+w],1,[[m,1]]);rec(B['plate_'+w],1,[[m,2]]);rec(B['sign_'+w],3,[[m,6],[I.stick,1]]);rec(B['hanging_sign_'+w],6,[[B[w+'_stripped_log'],6],[I.iron_ingot,2]])}
rec(B.bamboo_mosaic,1,[[SLAB[B.bamboo_planks],2]]);rec(B['slab_'+B.bamboo_mosaic],6,[[B.bamboo_mosaic,3]]);rec(B['stair_'+B.bamboo_mosaic],4,[[B.bamboo_mosaic,6]]);
for(const w in DW){const pl=w=='oak'?9:B[w+'_planks'];rec(I['door_'+w],3,[[pl,6]]);rec(B['trap_'+w],2,[[pl,6]])}
rec(I.door_iron,3,[[I.iron_ingot,6]]);rec(B.trap_iron,1,[[I.iron_ingot,4]]);rec(B.wall_cobble,6,[[4,6]]);
rec(B.ladder,3,[[I.stick,7]]);rec(B.chest,1,[['planks',8]]);rec(B.bed,1,[[B.wool_white,3],['planks',3]]);
rec(B.glass_pane,16,[[19,6]]);rec(B.iron_bars,16,[[I.iron_ingot,6]]);
rec(B.lever,1,[[I.stick,1],['cobble',1]]);rec(B.button,1,[[3,1]]);rec(B.plate,1,[[3,2]]);
rec(B.rtorch,1,[[I.stick,1],[I.redstone,1]]);rec(B.lamp,1,[[I.redstone,4],[25,1]]);rec(B.piston,1,[['planks',3],['cobble',4],[I.iron_ingot,1],[I.redstone,1]]);
rec(I.bucket,1,[[I.iron_ingot,3]]);rec(I.flint_steel,1,[[I.iron_ingot,1],[I.flint,1]]);rec(I.shears,1,[[I.iron_ingot,2]]);
rec(I.ender_eye,1,[[I.ender_pearl,1],[I.blaze_rod,1]]);
for(const w of Object.keys(WOODS).filter(w=>!['bamboo','crimson','warped'].includes(w)))rec(I['boat_'+w],1,[[B[w+'_planks']||9,5]]);rec(I.raft_bamboo,1,[[B.bamboo_planks,5]]);
 
/* ================= TEXTURAS EXTRAS (rostos) ================= */
tile('eface',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,15+rnd()*8,10,20);for(const[a,b,c,d]of[[2,7,5,8],[10,7,13,8]])for(let y=b;y<=d;y++)for(let x=a;x<=c;x++)p(x,y,230,60,230)});
tile('slface',p=>{for(let y=0;y<16;y++)for(let x=0;x<16;x++)p(x,y,90+rnd()*20,200+rnd()*20,80+rnd()*20,230);for(const[a,b,c,d]of[[3,5,5,7],[10,5,12,7],[7,10,8,11]])for(let y=b;y<=d;y++)for(let x=a;x<=c;x++)p(x,y,20,60,20)});
uploadAtlas();gl.activeTexture(gl.TEXTURE0);
// Cada muda possui seu próprio tile. Antes, várias apontavam para um TN inexistente e ficavam brancas.
for(const [w,v] of Object.entries(WOODS)){if(['crimson','warped'].includes(w))continue;const c=v[2];tile('sapling_'+w,p=>{clr(p);for(let y=7;y<16;y++){p(7,y,105,75,38);p(8,y,125,90,45)}for(let y=1;y<10;y++)for(let x=3;x<13;x++){const d=Math.hypot((x-8)*.9,y-5);if(d<4.4&&rnd()>.12)p(x,y,c[0]*.55+rnd()*25,c[1]*.75+rnd()*35,c[2]*.55+rnd()*20,255)}},w+'_sapling')}

 
/* ================= RUÍDO / TERRENO ================= */
const imul=Math.imul;
function hs(x,y,z){let h=imul(x|0,374761393)^imul(y|0,668265263)^imul(z|0,1442695041)^imul(SEED,1274126177);h=imul(h^(h>>>13),1274126177);h^=h>>>16;return(h>>>0)/4294967296}
const sm=t=>t*t*(3-2*t),lp=(a,b,t)=>a+(b-a)*t;
function n2(x,y,s=0){const ix=Math.floor(x),iy=Math.floor(y),fx=sm(x-ix),fy=sm(y-iy);return lp(lp(hs(ix,iy,s),hs(ix+1,iy,s),fx),lp(hs(ix,iy+1,s),hs(ix+1,iy+1,s),fx),fy)}
function n3(x,y,z,s=0){const ix=Math.floor(x),iy=Math.floor(y),iz=Math.floor(z),fx=sm(x-ix),fy=sm(y-iy),fz=sm(z-iz),a=ix+s*57;const g=(i,j,k)=>hs(a+i,iy+j,iz+k);
 return lp(lp(lp(g(0,0,0),g(1,0,0),fx),lp(g(0,1,0),g(1,1,0),fx),fy),lp(lp(g(0,0,1),g(1,0,1),fx),lp(g(0,1,1),g(1,1,1),fx),fy),fz)}
function fbm(x,y,o,s){let a=1,f=1,t=0,m=0;for(let i=0;i<o;i++){t+=n2(x*f,y*f,s+i)*a;m+=a;a*=.5;f*=2}return t/m}
const BIOMES=[
'Planícies','Planícies de girassóis','Planícies nevadas','Picos de gelo','Deserto','Pântano','Pântano de mangue','Floresta','Floresta de flores','Floresta de bétulas','Floresta de bétulas antigas','Floresta escura','Jardim pálido','Floresta salpicada','Bosque de cerejeiras','Taiga','Taiga nevada','Taiga de pinheiros antigos','Taiga de abetos antigos','Selva','Selva esparsa','Selva de bambu','Savana','Planalto de savana','Savana ventosa','Pradaria','Encostas nevadas','Picos congelados','Picos pontiagudos','Picos rochosos','Colinas ventosas','Colinas ventosas de cascalho','Floresta ventosa','Terras áridas','Terras áridas erodidas','Terras áridas arborizadas','Praia','Praia nevada','Costa pedregosa','Rio','Rio congelado','Oceano','Oceano profundo','Oceano morno','Oceano morno profundo','Oceano tépido','Oceano tépido profundo','Oceano frio','Oceano frio profundo','Oceano congelado','Oceano congelado profundo','Campos de cogumelos','Cavernas exuberantes','Cavernas de estalactites','Escuridão profunda','Cavernas de enxofre',
'Ermos do Nether','Floresta carmesim','Floresta distorcida','Vale da areia das almas','Deltas de basalto',
'Ilha central do End','Terras áridas do End','Terras médias do End','Terras altas do End','Pequenas ilhas do End','O Vazio'
];
const BIO={nether:56,end:61,void:66};
function col(wx,wz){
 const e=fbm(wx/260,wz/260,4,1),mt=fbm(wx/190,wz/190,3,5),tp=fbm(wx/520+9,wz/520,3,9),hu=fbm(wx/430,wz/430+4,3,13),cl=n2(wx/280,wz/280,121);
 let h=SEA+5+(e-.5)*50+(fbm(wx/18,wz/18,2,17)-.5)*5;
 if(e>.55)h+=Math.max(0,mt-.45)*Math.min(1,(e-.55)*6)*80;
 h=Math.max(3,Math.min(H-6,Math.floor(h)));
 let b;if(h<SEA)b=0;else if(h<=SEA+1)b=6;else if(h>46)b=5;else if(tp<.38)b=4;else if(tp>.62&&hu<.5)b=3;else if(hu>.55)b=2;else b=1;
 const rare=(scale,seed,threshold)=>n2(wx/scale,wz/scale,seed)>threshold;
 let bi=0;
 if(b==0){const warm=tp>.72,cold=tp<.25,deep=mt>.56;bi=warm?(deep?44:43):cold?(deep?50:49):tp>.5?(deep?46:45):tp>.4?(deep?42:41):(deep?48:47)}
 else if(b==6)bi=h>45?38:(tp<.3?37:36);
 else if(b==3)bi=tp<.15?34:tp>.85?35:tp>.55?4:33;
 else if(b==4)bi=tp<.2?3:(hu>.9?17:hu>.78?18:hu>.58?16:(mt>.7?2:15));
 else if(b==5)bi=h>65?(tp<.25?27:tp>.9?29:28):(tp<.3?26:(tp>.9?32:tp>.7?31:30));
 else if(b==2){
  if(tp<.04)bi=21;else if(tp<.10)bi=20;else if(tp<.16)bi=19;else if(tp<.19)bi=14;else if(tp<.22)bi=13;else if(tp<.24)bi=12;else if(tp<.36)bi=10;else if(tp<.51)bi=11;else if(tp<.65)bi=8;else if(tp<.82)bi=9;else bi=7;
  if(hu<.12)bi=5;else if(hu>.94&&rare(125,1008,.975))bi=6;
 }
 else {if(tp<.12)bi=1;else if(tp<.24)bi=22;else if(tp<.35)bi=23;else if(tp<.45)bi=24;else if(tp<.55)bi=25;else if(tp<.64)bi=0;else if(tp<.72)bi=7;else if(tp<.8)bi=7;else if(tp<.88)bi=hu<.3?40:39;else bi=0}
 // Biomas especiais são bolsões raros, em vez de aparecerem como escolhas climáticas comuns.
 if(b==1&&rare(115,1001,.972)&&hu>.43)bi=1; // planícies de girassóis
 if(b==2&&rare(125,1002,.982)&&tp<.5)bi=3; // picos de gelo
 if((bi==7||bi==8||bi==9||bi==10||bi==11)&&rare(145,1003,.988))bi=12; // jardim pálido, muito raro
 else if((bi==7||bi==8||bi==9||bi==10)&&rare(155,1004,.974)&&hu>.48)bi=14; // bosque de cerejeiras
 if((bi==41||bi==42||bi==43||bi==44||bi==45||bi==46||bi==47||bi==48||bi==49||bi==50)&&rare(240,1005,.988))bi=51; // campos de cogumelos
 if(bi==33&&rare(130,1006,.98))bi=34; // terras áridas erodidas
 if(bi==22&&rare(150,1007,.975))bi=24; // savana ventosa
 return[h,b,tp,bi]}

const cave=(x,y,z)=>(Math.abs(n3(x/22,y/11,z/22,31)-.5)<.055&&Math.abs(n3(x/22+40,y/11+40,z/22,37)-.5)<.055)||(y<40&&n3(x/26,y/14,z/26,41)>.75);
const mkD=()=>({chunks:new Map(),mods:new Map(),conts:new Map(),rss:new Set(),flq:new Set(),tks:new Set()});
const DIMS=[mkD(),mkD(),mkD()];
let dim=0,chunks=DIMS[0].chunks,mods=DIMS[0].mods,conts=DIMS[0].conts;
const lightBlock=t=>t&&BL[t].s&&!BL[t].tr&&!BL[t].leaf;
// Modelo de luz por voxel: níveis inteiros 0..15, como no Minecraft.
// A propagação é feita em lotes pequenos por frame para não bloquear o jogo.
const LIGHT_DIRS=[[1,0,0],[-1,0,0],[0,1,0],[0,-1,0],[0,0,1],[0,0,-1]];
const lightWorkChunks=[[],[],[]],lightResetKeys=[new Set(),new Set(),new Set()],lightResetJobs=[null,null,null];

// Atenuação de luz dos blocos: valores do Minecraft Bedrock 1.21.0.
// 0 = não atenua; 1-14 = atenua parcialmente; 15 = bloqueia.
// O padrão oficial é 15.

for(const b of BL){
 if(b)b.lightDampening=15;
}

const setLightDampening=(ids,value)=>{
 for(const id of ids){
  if(id!==undefined&&BL[id]){
   BL[id].lightDampening=value;
  }
 }
};

// Blocos com atenuação 0.
setLightDampening([
 0,14,19,
 B.glass_pane,
 B.iron_bars,
 B.sapling,
 B.bamboo_shoot,
 B.torch,
 B.tallgrass,
 B.dandelion,
 B.poppy,
 B.deadbush,
 B.bed,
 B.chest,
 B.farmland,
 B.wheat,
 B.rs_wire,
 B.lever,
 B.button,
 B.plate,
 B.rtorch,
 B.rtorch_off,
 B.portal,
 B.ladder,
 B.pistonhead,
 B.wall_cobble,
 B.wall_mossy
],0);

// Todas as folhas: valor 1.
for(const k of Object.keys(B)){
 if(k.endsWith('_leaves')){
  setLightDampening([B[k]],1);
 }
}

// Água: valor 2.
setLightDampening([10],2);

// Gelo normal: valor 3.
setLightDampening([13],3);

// Mudas e famílias de blocos transparentes.
setLightDampening(
 Object.keys(B)
  .filter(k=>k.endsWith('_sapling'))
  .map(k=>B[k]),
 0
);

// Portas, alçapões, cercas, portões, botões,
// placas e placas suspensas de madeira.
for(const k of Object.keys(B)){
 if(/^(fence_|gate_|button_|plate_|sign_|hanging_sign_|door_|trap_)/.test(k)){
  setLightDampening([B[k]],0);
 }
}

// Todas as lajes e escadas existentes.
setLightDampening(
 [...Object.values(SLAB),...Object.values(STAIR)],
 0
);

// Atenuação oficial consultada pelo sistema de iluminação.
function lightOpacity(id,kind=0){
 if(!id)return 0;
 const b=BL[id];
 if(!b)return 15;
 return b.lightDampening;
}
function lightEmission(id){
 if(!id)return 0;
 if(id==26)return 15;                         // lava
 if(id==25||id==B.sea_lantern||id==B.lamp_on)return 15;
 if(id==B.torch)return 14;
 if(id==B.furnace_on)return 13;
 if(id==B.portal)return 11;
 if(id==B.rtorch)return 7;
 if(id==B.magma)return 3;
 const b=BL[id];return b&&(b.emit||b.em)?(b.lightLevel||b.emission||12):0;
}
function enqueueLight(c,i,kind){
 if(!c||!c.sl||i<0||i>=c.d.length)return;
 if(!c.lqs)c.lqs=new Uint8Array(c.d.length);
 if(!c.lqb)c.lqb=new Uint8Array(c.d.length);
 const flags=kind?c.lqb:c.lqs;if(flags[i])return;flags[i]=1;
 if(!c.lightQueue)c.lightQueue=[];
 c.lightQueue.push((i<<1)|(kind?1:0));
 if(!c.lightQueued){c.lightQueued=true;lightWorkChunks[dim].push(c)}
}
function prepareChunkLightData(c){
 const n=c.d.length;
 if(!c.sl||c.sl.length!==n)c.sl=new Uint8Array(n);else c.sl.fill(0);
 if(!c.bl||c.bl.length!==n)c.bl=new Uint8Array(n);else c.bl.fill(0);
 if(!c.lqs||c.lqs.length!==n)c.lqs=new Uint8Array(n);else c.lqs.fill(0);
 if(!c.lqb||c.lqb.length!==n)c.lqb=new Uint8Array(n);else c.lqb.fill(0);
 c.lightQueue=[];c.lightHead=0;c.lightQueued=false;c.lightPending=true;
 // Luz solar direta: 15 no céu aberto, atenuada por água e folhas, bloqueada por cubos opacos.
 if(dim!==1)for(let z=0;z<16;z++)for(let x=0;x<16;x++){
  let level=15;
  for(let y=H-1;y>=0;y--){const i=x+16*(z+16*y),op=lightOpacity(c.d[i],0);
   if(op>=15)level=0;else if(op>0)level=Math.max(0,level-op);
   c.sl[i]=level;
  }
 }
 for(let i=0;i<n;i++){const em=lightEmission(c.d[i]);if(em)c.bl[i]=em}
 c.dirty=true;
}
function lightNeighbor(c,i,dx,dy,dz){
 const x=i&15,z=(i>>4)&15,y=i>>8,nx=c.cx*16+x+dx,nz=c.cz*16+z+dz,ny=y+dy;
 if(ny<0||ny>=H)return null;
 const nc=chunkAt(nx,nz);if(!nc||!nc.sl)return null;
 const ni=(nx&15)+16*((nz&15)+16*ny);return[nc,ni];
}
function seedChunkLight(c){
 const skyMap=c.sl,blockMap=c.bl,blocks=c.d;
 for(let y=0;y<H;y++)for(let z=0;z<16;z++)for(let x=0;x<16;x++){
  const i=x+16*(z+16*y),sky=skyMap[i];
  if(sky>1){let edge=false;
   if(x>0){const j=i-1,op=lightOpacity(blocks[j]);if(op<15&&sky-Math.max(1,op)>skyMap[j])edge=true}
   if(!edge&&x<15){const j=i+1,op=lightOpacity(blocks[j]);if(op<15&&sky-Math.max(1,op)>skyMap[j])edge=true}
   if(!edge&&z>0){const j=i-16,op=lightOpacity(blocks[j]);if(op<15&&sky-Math.max(1,op)>skyMap[j])edge=true}
   if(!edge&&z<15){const j=i+16,op=lightOpacity(blocks[j]);if(op<15&&sky-Math.max(1,op)>skyMap[j])edge=true}
   if(edge)enqueueLight(c,i,0);
  }
  if(blockMap[i]>1)enqueueLight(c,i,1);
 }
}
function seedBorderPair(a,b,dx,dz){
 if(!a||!b||!a.sl||!b.sl)return;
 for(let y=0;y<H;y++)for(let t=0;t<16;t++){
  let ax,az,bx,bz;
  if(dx){ax=dx>0?15:0;az=t;bx=dx>0?0:15;bz=t}
  else{ax=t;az=dz>0?15:0;bx=t;bz=dz>0?0:15}
  const ai=ax+16*(az+16*y),bi=bx+16*(bz+16*y);
  for(let kind=0;kind<2;kind++){
   const am=kind?a.bl:a.sl,bm=kind?b.bl:b.sl,al=am[ai],bl=bm[bi];
   if(al>1&&lightOpacity(b.d[bi],kind)<15&&al-Math.max(1,lightOpacity(b.d[bi],kind))>bl)enqueueLight(a,ai,kind);
   if(bl>1&&lightOpacity(a.d[ai],kind)<15&&bl-Math.max(1,lightOpacity(a.d[ai],kind))>al)enqueueLight(b,bi,kind);
  }
 }
}
function initChunkLighting(c){
 prepareChunkLightData(c);seedChunkLight(c);
 for(const[dx,dz]of[[1,0],[-1,0],[0,1],[0,-1]]){const n=chunks.get((c.cx+dx)+','+(c.cz+dz));if(n)seedBorderPair(c,n,dx,dz)}
 c.lightPending=false;
}

function scheduleLightingReset(cx,cz){
 lightResetKeys[dim].add(cx+','+cz);

 for(let dz=-1;dz<=1;dz++)for(let dx=-1;dx<=1;dx++){
  const c=chunks.get((cx+dx)+','+(cz+dz));
  if(c)c.lightPending=true;
 }
}

function beginLightingReset(){
 const keys=lightResetKeys[dim];
 if(lightResetJobs[dim]||!keys.size)return;

 const first=keys.values().next().value;
 keys.delete(first);

 const[cx,cz]=first.split(',').map(Number);
 const region=[];

 // A região cobre a distância máxima de propagação da luz.
 for(let dz=-1;dz<=1;dz++)for(let dx=-1;dx<=1;dx++){
  const k=(cx+dx)+','+(cz+dz);
  const c=chunks.get(k);

  if(c){
   region.push(c);
   keys.delete(k);
  }
 }

 if(!region.length)return;

 const ids=new Map();
 const work=lightWorkChunks[dim];

 for(let i=work.length-1;i>=0;i--){
  if(region.includes(work[i]))work.splice(i,1);
 }

 for(let ci=0;ci<region.length;ci++){
  const c=region[ci];
  ids.set(c,ci);

  // Buffers novos. c.sl e c.bl continuam sendo usados na tela.
  c.nextSl=new Uint8Array(c.d.length);
  c.nextBl=new Uint8Array(c.d.length);
  c.nqs=new Uint8Array(c.d.length);
  c.nqb=new Uint8Array(c.d.length);

  c.lightQueue=[];
  c.lightHead=0;
  c.lightQueued=false;
  c.lightPending=true;

  if(c.lqs)c.lqs.fill(0);
  if(c.lqb)c.lqb.fill(0);
 }

 lightResetJobs[dim]={
  region,ids,
  stage:0,
  chunk:0,
  column:0,
  index:0,
  queue:[],
  head:0
 };
}

function lightJobPush(j,ci,i,kind){
 const c=j.region[ci];
 const flags=kind?c.nqb:c.nqs;

 if(flags[i])return;

 flags[i]=1;
 j.queue.push((((ci*c.d.length+i)<<1)|kind));
}

function processLightingReset(){
 if(!lightResetJobs[dim])beginLightingReset();

 const j=lightResetJobs[dim];
 if(!j)return;

 // FASE 1: calcular a luz solar direta.
 if(j.stage===0){
  let budget=128;

  while(budget-->0&&j.chunk<j.region.length){
   const c=j.region[j.chunk];
   const x=j.column&15;
   const z=j.column>>4;

   let level=dim===1?0:15;

   for(let y=H-1;y>=0;y--){
    const i=x+16*(z+16*y);
    const op=lightOpacity(c.d[i],0);

    if(op>=15)level=0;
    else if(op>0)level=Math.max(0,level-op);

    c.nextSl[i]=level;
   }

   j.column++;

   if(j.column>=256){
    j.column=0;
    j.chunk++;
   }
  }

  if(j.chunk>=j.region.length){
   j.stage=1;
   j.chunk=0;
   j.index=0;
  }

  return;
 }

 // FASE 2: identificar fontes e pontos de propagação.
 if(j.stage===1){
  let budget=12000;

  while(budget-->0&&j.chunk<j.region.length){
   const c=j.region[j.chunk];
   const ci=j.chunk;
   const i=j.index++;

   if(i>=c.d.length){
    j.chunk++;
    j.index=0;
    continue;
   }

   const x=i&15;
   const z=(i>>4)&15;
   const y=i>>8;
   const wx=c.cx*16+x;
   const wz=c.cz*16+z;

   const emission=lightEmission(c.d[i]);

   if(emission){
    c.nextBl[i]=emission;
    lightJobPush(j,ci,i,1);
   }

   for(let kind=0;kind<2;kind++){
    const map=kind?c.nextBl:c.nextSl;
    const level=map[i];

    if(level<=1)continue;

    let canSpread=false;

    for(const d of LIGHT_DIRS){
     const nx=wx+d[0];
     const ny=y+d[1];
     const nz=wz+d[2];

     if(ny<0||ny>=H)continue;

     const nc=chunkAt(nx,nz);
     if(!nc||!nc.sl)continue;

     const ni=(nx&15)+16*((nz&15)+16*ny);
     const op=lightOpacity(nc.d[ni],kind);

     if(op>=15)continue;

     const next=level-Math.max(1,op);
     const nci=j.ids.get(nc);

     if(nci!==undefined){
      const target=kind?nc.nextBl:nc.nextSl;
      if(next>target[ni])canSpread=true;
     }else{
      const external=kind?nc.bl[ni]:nc.sl[ni];
      const candidate=external-Math.max(1,lightOpacity(c.d[i],kind));

      if(candidate>map[i]){
       map[i]=candidate;
       canSpread=true;
      }
     }
    }

    if(canSpread)lightJobPush(j,ci,i,kind);
   }
  }

  if(j.chunk>=j.region.length){
   j.stage=2;
   j.head=0;
  }

  return;
 }

 // FASE 3: propagar luz dentro da região temporária.
 if(j.stage===2){
  let budget=10000;

  while(budget-->0&&j.head<j.queue.length){
   const code=j.queue[j.head++];
   const p=code>>1;
   const kind=code&1;
   const n=j.region[0].d.length;

   const ci=Math.floor(p/n);
   const i=p-ci*n;
   const c=j.region[ci];

   (kind?c.nqb:c.nqs)[i]=0;

   const map=kind?c.nextBl:c.nextSl;
   const level=map[i];

   if(level<=1)continue;

   const x=i&15;
   const z=(i>>4)&15;
   const y=i>>8;
   const wx=c.cx*16+x;
   const wz=c.cz*16+z;

   for(const d of LIGHT_DIRS){
    const nx=wx+d[0];
    const ny=y+d[1];
    const nz=wz+d[2];

    if(ny<0||ny>=H)continue;

    const nc=chunkAt(nx,nz);
    if(!nc||!nc.sl)continue;

    const nci=j.ids.get(nc);
    if(nci===undefined)continue;

    const ni=(nx&15)+16*((nz&15)+16*ny);
    const op=lightOpacity(nc.d[ni],kind);

    if(op>=15)continue;

    const next=level-Math.max(1,op);
    const target=kind?nc.nextBl:nc.nextSl;

    if(next>target[ni]){
     target[ni]=next;
     lightJobPush(j,nci,ni,kind);
    }
   }
  }

  if(j.head>=j.queue.length){
   // Reinicia somente se houve outra edição dentro desta região.
   const changedAgain=j.region.some(c=>
    lightResetKeys[dim].has(c.cx+','+c.cz)
   );

   if(changedAgain){
    for(const c of j.region){
     c.nextSl=null;
     c.nextBl=null;
     c.nqs=null;
     c.nqb=null;
     c.lightPending=false;
    }

    lightResetJobs[dim]=null;
    beginLightingReset();
    return;
   }

   // FASE 4: troca os mapas de uma só vez.
   for(const c of j.region){
    let changed=false;

    for(let i=0;i<c.d.length;i++){
     if(c.sl[i]!==c.nextSl[i]||c.bl[i]!==c.nextBl[i]){
      changed=true;
      break;
     }
    }

    c.sl=c.nextSl;
    c.bl=c.nextBl;

    c.nextSl=null;
    c.nextBl=null;
    c.nqs=null;
    c.nqb=null;

    c.lightQueue=[];
    c.lightHead=0;
    c.lightQueued=false;

    if(c.lqs)c.lqs.fill(0);
    if(c.lqb)c.lqb.fill(0);

    if(changed)c.dirty=true;
    c.lightPending=false;
   }

   lightResetJobs[dim]=null;
  }
 }
}

function processLighting(budget=2500){
 if(lightResetJobs[dim])return;

 const work=lightWorkChunks[dim];

 while(budget>0&&work.length){
  const c=work.shift();

  if(chunks.get(c.cx+','+c.cz)!==c){
   c.lightQueue=[];
   c.lightHead=0;
   c.lightQueued=false;
   continue;
  }

  const start=c.lightHead;
  const end=Math.min(
   c.lightQueue.length,
   start+Math.min(96,budget)
  );

  while(c.lightHead<end){
   const code=c.lightQueue[c.lightHead++];
   const kind=code&1;
   const i=code>>1;

   (kind?c.lqb:c.lqs)[i]=0;

   const level=(kind?c.bl:c.sl)[i];
   if(level<=1)continue;

   const x=i&15;
   const z=(i>>4)&15;
   const y=i>>8;
   const wx=c.cx*16+x;
   const wz=c.cz*16+z;

   for(const d of LIGHT_DIRS){
    const nx=wx+d[0];
    const ny=y+d[1];
    const nz=wz+d[2];

    if(ny<0||ny>=H)continue;

    const nc=chunkAt(nx,nz);
    if(!nc||!nc.sl)continue;

    const ni=(nx&15)+16*((nz&15)+16*ny);
    const op=lightOpacity(nc.d[ni],kind);

    if(op>=15)continue;

    const next=level-Math.max(1,op);
    const dst=kind?nc.bl:nc.sl;

    if(next>dst[ni]){
     dst[ni]=next;
     nc.dirty=true;
     enqueueLight(nc,ni,kind);
    }
   }
  }

  budget-=end-start;

  if(c.lightHead>=c.lightQueue.length){
   c.lightQueue=[];
   c.lightHead=0;
   c.lightQueued=false;
  }else{
   work.push(c);
  }
 }
}

function lightCell(x,y,z){
 if(y<0)return[0,0];if(y>=H)return[dim===1?0:1,0];
 const c=chunkAt(x,z);if(!c||!c.sl)return[0,0];const i=(x&15)+16*((z&15)+16*y);
 return[c.sl[i]/15,c.bl[i]/15];
}
function genChunk(cx,cz){
 const d=new Uint16Array(256*H),mt_=new Uint8Array(256*H),hm=new Uint8Array(256),bio=new Uint8Array(256);let top=0;
 if(dim)top=genDim(d,bio,cx,cz);
 for(let z=0;z<(dim?0:16);z++)for(let x=0;x<16;x++){
  const wx=cx*16+x,wz=cz*16+z,[h,b,tp,bi]=col(wx,wz);bio[x+z*16]=bi;
  for(let y=0;y<=Math.max(h,SEA);y++){
   let id=0;
   if(y==0)id=11;
   else if(y<=h){const dp=h-y;
    if(b==0||b==6){id=dp<3?(b==6||n2(wx/25,wz/25,3)<.55?5:6):3;if(b==0&&dp<3&&n2(wx/9,wz/9,5)>.74)id=B.clay}
    else if(b==3)id=dp<4?5:dp<7?21:3;
    else if(dp==0)id=b==4?12:(b==5?(h>58?27:(h>50?3:1)):(b==2&&n2(wx/9,wz/9,8)>.75?B.podzol:1));
    else if(dp<4)id=(b==5&&h>50)?3:2;
    else id=3;
    if(id==3){const r=hs(wx,y,wz^555);
     if(r<.0025&&y<14)id=18;else if(r<.005&&y<16)id=B.redstone_ore;else if(r<.0075&&y<28)id=17;else if(r<.0095&&y<30)id=B.lapis_ore;
     else if(b==5&&r<.0115&&y<40)id=B.emerald_ore;else if(r<.025&&y<48)id=16;else if(r<.035&&y<56)id=B.copper_ore;else if(r<.055&&y<64)id=15;
     else if(n3(wx/12,y/12,wz/12,61)>.8)id=B.granite;else if(n3(wx/12,y/12,wz/12,67)>.8)id=B.diorite;else if(n3(wx/12,y/12,wz/12,71)>.8)id=B.andesite;
     if(id==3&&y<9)id=B.deepslate}
    if(y>2&&(b==5?dp>=1:dp>=3)&&cave(wx,y,wz))id=y<7?26:0;
   }else if(y<=SEA)id=(y==SEA&&tp<.35&&b==0)?13:10;
   d[x+16*(z+16*y)]=id;if(id&&y>top)top=y}
  const sI=x+16*(z+16*(h+1));
  if(h>SEA+1&&h+2<H&&!d[sI]){const r2=hs(wx,wz,71),g=d[x+16*(z+16*h)];
   if(g==1&&(b==1||b==2||b==5&&h<50)){if(r2<.1)d[sI]=B.tallgrass;else if(r2<.11&&b!=5)d[sI]=B.dandelion;else if(r2<.12&&b!=5)d[sI]=B.poppy;if(d[sI]&&h+1>top)top=h+1}
   else if(b==3&&g==5&&r2<.01){d[sI]=B.deadbush;top=Math.max(top,h+1)}}
  if(b==3&&h>SEA+1&&hs(wx,wz,21)<.012){const n=1+(hs(wx,wz,22)*3|0);for(let k=1;k<=n;k++)d[x+16*(z+16*(h+k))]=14;top=Math.max(top,h+n)}
 }
 const put=(wx,y,wz,id,f)=>{const lx=wx-cx*16,lz=wz-cz*16;if(lx<0||lx>15||lz<0||lz>15||y<0||y>=H)return;const i=lx+16*(lz+16*y);if(f||!d[i]||BL[d[i]].cross){d[i]=id;if(y>top)top=y}};
 for(let tz=cz*16-2;tz<cz*16+(dim?-2:18);tz++)for(let tx=cx*16-2;tx<cx*16+18;tx++){
  const r=hs(tx,tz,51);if(r>.045)continue;
  const[h,b,tp,bi]=col(tx,tz);if(h<=SEA+1||b==0||b==3||b==6)continue;
  const p=bi==13?.055:bi==14?.04:bi==12?.035:bi==6?.035:(b==2?.045:b==1?.004:b==4?.008:b==5?(h<50?.01:0):0);if(r>=p)continue;
  const kind=bi==13?'poplar':bi==14?'cherry':bi==12?'pale_oak':bi==6?'mangrove':bi==9||bi==10?'birch':bi==15||bi==16||bi==17||bi==18?'spruce':bi==19||bi==20||bi==21?'jungle':(b==4||b==5?'spruce':(b==2&&hs(tx,tz,53)<.4?'birch':'oak'));
  const lg=B[kind+'_log'],lf=kind=='poplar'?[B.poplar_red_leaves,B.poplar_orange_leaves,B.poplar_yellow_leaves][Math.floor(hs(tx,tz,124)*3)]:B[kind+'_leaves'];
  const th=(kind=='spruce'||kind=='poplar'?5:4)+(hs(tx,tz,52)*3|0);
  for(let k=1;k<=th;k++)put(tx,h+k,tz,lg,true);
  for(let dy=th-2;dy<=th+1;dy++){const rad=dy>=th?1:2;
   for(let dz=-rad;dz<=rad;dz++)for(let dx=-rad;dx<=rad;dx++){
    if(Math.abs(dx)==rad&&Math.abs(dz)==rad&&(dy>=th||hs(tx+dx,dy,tz+dz)<.5))continue;
    if(dx==0&&dz==0&&dy<=th)continue;put(tx+dx,h+dy,tz+dz,lf,false)}}
 }
 const c={cx,cz,d,m:mt_,hm,bio,cbio:new Uint8Array(256*H),top,dirty:true,em:[],mo:null,mt:null};
 if(dim)for(let i=0;i<d.length;i++){const q=d[i];if(q&&BL[q].emit)c.em.push([cx*16+(i&15),i>>8,cz*16+((i>>4)&15)])}
 const m=mods.get(cx+','+cz);
 if(m)for(const[i,v]of m){const id=v&4095;d[i]=id;mt_[i]=v>>12;const y=(i>>8);if(y>c.top)c.top=y;if(BL[id]&&BL[id].emit)c.em.push([cx*16+(i&15),y,cz*16+((i>>4)&15)]);rsReg(cx*16+(i&15),y,cz*16+((i>>4)&15),id)}
 for(let z=0;z<16;z++)for(let x=0;x<16;x++){let y=c.top;while(y>0&&!lightBlock(d[x+16*(z+16*y)]))y--;hm[x+z*16]=y}
 for(let z=0;z<16;z++)for(let x=0;x<16;x++){
  const wx=cx*16+x,wz=cz*16+z,surf=bio[x+z*16];
  for(let y=0;y<H;y++){
   let bi=surf;
   if(!dim&&y<hm[x+z*16]-3&&d[x+16*(z+16*y)]==0){
    const caveWet=n2(wx/34,wz/34,202),caveRock=n2(wx/27,wz/27,203),deep=n3(wx/28,y/18,wz/28,204);
    if(y<22&&deep>.61)bi=54;else if(y<40&&caveRock>.72)bi=55;else if(caveWet>.62)bi=52;else if(caveRock>.5)bi=53;
   }
   c.cbio[x+16*(z+16*y)]=bi;
  }
 }
 return c}
function getBlock(x,y,z){if(y<0)return 3;if(y>=H)return 0;const c=chunks.get((x>>4)+','+(z>>4));return c?c.d[(x&15)+16*((z&15)+16*y)]:0}
const chunkAt=(x,z)=>chunks.get((Math.floor(x)>>4)+','+(Math.floor(z)>>4));
function dirtyC(cx,cz){const c=chunks.get(cx+','+cz);if(c)c.dirty=true}
function setBlock(x,y,z,id,meta=0){
 if(y<0||y>=H)return;const cx=x>>4,cz=z>>4,k=cx+','+cz,c=chunks.get(k);if(!c)return;
 const lx=x&15,lz=z&15,i=lx+16*(lz+16*y),old=c.d[i],om=c.m[i];if(old===id&&om===meta)return;c.d[i]=id;c.m[i]=meta;rsReg(x,y,z,id);fluidNote(x,y,z);
 let m=mods.get(k);if(!m)mods.set(k,m=new Map());m.set(i,id|(meta<<12));if(y>c.top)c.top=y;
 const hi=c.hm[lx+lz*16],pass=lightBlock(id);
 if(pass&&y>hi)c.hm[lx+lz*16]=y;else if(!pass&&y==hi){let yy=y-1;while(yy>0&&!lightBlock(c.d[lx+16*(lz+16*yy)]))yy--;c.hm[lx+lz*16]=yy}
 c.dirty=true;if(lx==0)dirtyC(cx-1,cz);if(lx==15)dirtyC(cx+1,cz);if(lz==0)dirtyC(cx,cz-1);if(lz==15)dirtyC(cx,cz+1);
 if((old&&BL[old].emit)||(id&&BL[id].emit)){c.em=c.em.filter(e=>!(e[0]==x&&e[1]==y&&e[2]==z));if(id&&BL[id].emit)c.em.push([x,y,z]);for(let a=-1;a<=1;a++)for(let b=-1;b<=1;b++)dirtyC(cx+a,cz+b)}
 if(lightOpacity(old,0)!==lightOpacity(id,0)||lightOpacity(old,1)!==lightOpacity(id,1)||lightEmission(old)!==lightEmission(id))scheduleLightingReset(cx,cz)}
const hasNb=c=>chunks.has((c.cx-1)+','+c.cz)&&chunks.has((c.cx+1)+','+c.cz)&&chunks.has(c.cx+','+(c.cz-1))&&chunks.has(c.cx+','+(c.cz+1));
function edit(x,y,z,id,meta=0){setBlock(x,y,z,id,meta);const cx=x>>4,cz=z>>4;for(let a=-1;a<=1;a++)for(let b=-1;b<=1;b++){const c=chunks.get((cx+a)+','+(cz+b));if(c&&c.dirty&&hasNb(c))meshChunk(c)}}
 
/* ================= MESH ================= */
const DIRS=[[1,0,0],[-1,0,0],[0,1,0],[0,-1,0],[0,0,1],[0,0,-1]];

function vertexLight(x,y,z,face,corner,readLight=lightCell,readOpaque=null){
 const d=DIRS[face],base=[x+d[0],y+d[1],z+d[2]],tang=face<2?[1,2]:face<4?[0,2]:[0,1];
 const a=tang[0],b=tang[1],sa=corner[a]>=.5?1:-1,sb=corner[b]>=.5?1:-1;
 const cells=[base,base.slice(),base.slice(),base.slice()];
 cells[1][a]+=sa;
 cells[2][b]+=sb;
 cells[3][a]+=sa;
 cells[3][b]+=sb;

 let sky=0,block=0;
 for(const q of cells){
  const l=readLight(q[0],q[1],q[2]);
  sky+=l[0];
  block+=l[1];
 }

 const opaque=q=>{
  if(q[1]<0)return true;
  if(q[1]>=H)return false;
  const c=chunkAt(q[0],q[2]);
  if(!c)return false;
  const i=(q[0]&15)+16*((q[2]&15)+16*q[1]);
  return lightOpacity(c.d[i])>=15;
 };

 const opaqueAt=q=>readOpaque
  ?readOpaque(q[0],q[1],q[2])
  :opaque(q);

 const a1=opaqueAt(cells[1]);
 const a2=opaqueAt(cells[2]);
 const a3=opaqueAt(cells[3]);
 const ao=a1&&a2
  ? .58
  : 1-(Number(a1)+Number(a2)+Number(a3))*.14;

 return[sky*.25,block*.25,ao];
}

const FC=[[[1,0,1],[1,0,0],[1,1,0],[1,1,1]],[[0,0,0],[0,0,1],[0,1,1],[0,1,0]],[[0,1,1],[1,1,1],[1,1,0],[0,1,0]],[[0,0,0],[1,0,0],[1,0,1],[0,0,1]],[[0,0,1],[1,0,1],[1,1,1],[0,1,1]],[[1,0,0],[0,0,0],[0,1,0],[1,1,0]]];
const CROSS=[[[0,0,0],[1,0,1],[1,1,1],[0,1,0]],[[1,0,0],[0,0,1],[0,1,1],[1,1,0]]];
// Direções: leste, oeste, cima, baixo, sul, norte.
const SHD=[.68,.68,1,.5,.82,.82],UVC=[[0,1],[1,1],[1,0],[0,0]],AOF=[.5,.68,.84,1];
const TINT=Array.from({length:67},(_,i)=>i==4||i>=33&&i<=35?[.8,.72,.38]:i==2||i==3||i==16||i==26||i==27?[.86,.91,.98]:i==5||i==6?[.32,.58,.36]:i==12?[.76,.8,.74]:i==13?[.66,.63,.54]:i==14?[.95,.62,.72]:i>=15&&i<=18?[.42,.62,.38]:i>=19&&i<=21?[.34,.72,.24]:i>=56&&i<=60?[.65,.38,.3]:i>=61?[.5,.5,.5]:[.55,.82,.33]);
const tu=(t,u)=>((t&(TPR-1))+.004+u*.992)/TPR,tv=(t,v)=>((t/TPR|0)+.004+v*.992)/TPR;
function meshChunk(c){
 c.dirty=false;
 const nbs=[chunks.get((c.cx-1)+','+c.cz),chunks.get((c.cx+1)+','+c.cz),chunks.get(c.cx+','+(c.cz-1)),chunks.get(c.cx+','+(c.cz+1))];
 const gb=(x,y,z)=>{if(y<0)return 3;if(y>=H)return 0;if((x<0||x>15)&&(z<0||z>15))return 0;
  if(x<0){const n=nbs[0];return n?n.d[15+16*(z+16*y)]:3}if(x>15){const n=nbs[1];return n?n.d[16*(z+16*y)]:3}
  if(z<0){const n=nbs[2];return n?n.d[x+16*(15+16*y)]:3}if(z>15){const n=nbs[3];return n?n.d[x+256*y]:3}return c.d[x+16*(z+16*y)]};
 const ov=[],oi=[],tvv=[],ti=[];let on_=0,tn=0;const bx=c.cx*16,bz=c.cz*16;
 const lightGrid=c.lightGrid&&c.lightGrid.length===18*18*(H+2)*2?c.lightGrid:(c.lightGrid=new Uint8Array(18*18*(H+2)*2));
 const opacityGrid=c.opacityGrid&&c.opacityGrid.length===18*18*(H+2)?c.opacityGrid:(c.opacityGrid=new Uint8Array(18*18*(H+2)));
  for(let ly=-1;ly<=H;ly++)
 for(let lz=-1;lz<=16;lz++)
 for(let lx=-1;lx<=16;lx++){
  let sky=0,block=0,opaque=ly<0?1:0;

  if(ly>=H){
   sky=dim===1?0:15;
  }else if(ly>=0){
   const lc=chunkAt(bx+lx,bz+lz);
   if(lc&&lc.sl){
    const li=((bx+lx)&15)+16*(((bz+lz)&15)+16*ly);
    sky=lc.sl[li];
    block=lc.bl[li];
    opaque=lightOpacity(lc.d[li])>=15?1:0;
   }
  }

  const cell=(lx+1)+18*((lz+1)+18*(ly+1));
  const gi=cell*2;
  lightGrid[gi]=sky;
  lightGrid[gi+1]=block;
  opacityGrid[cell]=opaque;
 }
 const lightPair=[0,0],readLight=(wx,wy,wz)=>{const lx=wx-bx,lz=wz-bz;if(wy<0){lightPair[0]=lightPair[1]=0;return lightPair}if(wy>=H){lightPair[0]=dim===1?0:1;lightPair[1]=0;return lightPair}const gi=2*((lx+1)+18*((lz+1)+18*(wy+1)));lightPair[0]=lightGrid[gi]/15;lightPair[1]=lightGrid[gi+1]/15;return lightPair};
 const readOpaque=(wx,wy,wz)=>{if(wy<0)return true;if(wy>=H)return false;const cell=(wx-bx+1)+18*((wz-bz+1)+18*(wy+1));return opacityGrid[cell]===1};
 for(let y=0;y<=c.top;y++)for(let z=0;z<16;z++)for(let x=0;x<16;x++){
  const id=c.d[x+16*(z+16*y)];if(!id)continue;const b=BL[id],T=b.tr;
  const meta=c.m[x+16*(z+16*y)];
  if(b.box){const bxs=typeof b.box=='function'?b.box(meta,bx+x,y,bz+z):b.box,tf=b.tf?b.tf(meta):[1,1,1];
   const V=T?tvv:ov,I_=T?ti:oi;
   for(const q of bxs){for(let f=0;f<6;f++){const d=DIRS[f];
     if((f==0&&q[3]>=1)||(f==1&&q[0]<=0)||(f==2&&q[4]>=1)||(f==3&&q[1]<=0)||(f==4&&q[5]>=1)||(f==5&&q[2]<=0)){const nid=gb(x+d[0],y+d[1],z+d[2]);if(nid&&(nid==id||(BL[nid].s&&!BL[nid].tr&&!BL[nid].box&&!BL[nid].cross)))continue}
     const tl=q[6]!==undefined?q[6]:(b.faceTex?b.faceTex(meta,f,bx+x,y,bz+z):(b.ttop&&f==2?b.ttop(meta):(b.tmeta?b.tmeta(meta):(f==2?b.t[0]:f==3?b.t[1]:b.t[2])))),sh=SHD[f]*.92,base=T?tn:on_;
     const aoV=[];
     for(let k=0;k<4;k++){const cc=FC[f][k],px=cc[0]?q[3]:q[0],py=cc[1]?q[4]:q[1],pz=cc[2]?q[5]:q[2];let u,v;
      if(f==0){u=1-pz;v=1-py}else if(f==1){u=pz;v=1-py}else if(f==2){u=px;v=pz}else if(f==3){u=px;v=1-pz}else if(f==4){u=px;v=1-py}else{u=1-px;v=1-py}
      // Nos troncos horizontais, gira a casca para que os veios acompanhem o eixo real.
      const axis=meta&3,endFace=b.log&&(axis==0?(f==2||f==3):axis==1?(f==0||f==1):(f==4||f==5));if(b.log&&!endFace){if(axis==1){const oldU=u;u=v;v=oldU}else if(axis==2){const oldU=u;u=1-v;v=oldU}}
      const li=vertexLight(bx+x,y,bz+z,f,[px,py,pz],readLight,readOpaque),shade=sh*li[2];aoV.push(li[2]);
      V.push(bx+x+px,y+py,bz+z+pz,tu(tl,u),tv(tl,v),tf[0]*shade,tf[1]*shade,tf[2]*shade,li[0],li[1])}
     if(aoV[0]+aoV[2]>aoV[1]+aoV[3])I_.push(base,base+1,base+2,base,base+2,base+3);else I_.push(base+1,base+2,base+3,base+1,base+3,base);
     if(T)tn+=4;else on_+=4}}
   continue}
  if(b.cross){const tl=b.tmeta?b.tmeta(meta):b.t[2],tc_=b.tint?TINT[c.bio[x+z*16]]:[1,1,1],li=lightCell(bx+x,y,bz+z);
   for(const pl of CROSS){for(let k=0;k<4;k++){const q=pl[k];ov.push(bx+x+q[0],y+q[1],bz+z+q[2],tu(tl,UVC[k][0]),tv(tl,UVC[k][1]),tc_[0]*.9,tc_[1]*.9,tc_[2]*.9,li[0],li[1])}oi.push(on_,on_+1,on_+2,on_,on_+2,on_+3);on_+=4}continue}
  for(let f=0;f<6;f++){
   const d=DIRS[f],nx=x+d[0],ny=y+d[1],nz=z+d[2],nid=gb(nx,ny,nz);
   if(nid&&(nid==id||(BL[nid].s&&!BL[nid].tr&&!BL[nid].box)))continue;
   const axis=meta&3;
    const endFace=b.log&&(axis==0?(f==2||f==3):axis==1?(f==0||f==1):(f==4||f==5));
    const tl=b.log?(endFace?b.t[0]:b.t[2]):(f==2?b.t[0]:f==3?b.t[1]:b.t[2]);
   const tn_=(b.tint&&(f==2||b.leaf))?(b.tc||TINT[c.bio[x+z*16]]):[1,1,1];
   const V=T?tvv:ov,I_=T?ti:oi,base=T?tn:on_,up=(id==10||id==26)&&gb(x,y+1,z)!=id,fl=meta==8?0:Math.max(0,Math.min(.875,meta/8));
   const aoV=[];
   for(let k=0;k<4;k++){const cc=FC[f][k],li=vertexLight(bx+x,y,bz+z,f,cc,readLight,readOpaque),s=SHD[f]*li[2];aoV.push(li[2]);
    V.push(bx+x+cc[0],y+cc[1]-(up&&cc[1]?fl:0),bz+z+cc[2],tu(tl,UVC[k][0]),tv(tl,UVC[k][1]),tn_[0]*s,tn_[1]*s,tn_[2]*s,li[0],li[1])}
   if(aoV[0]+aoV[2]>aoV[1]+aoV[3])I_.push(base,base+1,base+2,base,base+2,base+3);else I_.push(base+1,base+2,base+3,base+1,base+3,base);
   if(T)tn+=4;else on_+=4}}
 freeMesh(c.mo);freeMesh(c.mt);c.mo=mkMesh(ov,oi);c.mt=mkMesh(tvv,ti)}
 
/* ================= ESTADO ================= */
const P={x:.5,y:80,z:.5,vx:0,vy:0,vz:0,yaw:0,pitch:0,w:.6,h:1.8,onGround:false,hp:20,food:20,air:10,fly:false,sprint:false,peak:80,mine:null,target:null,walkT:0,walkAmp:0,fx:{}};
let gm=1,dead=false,peace=false,tod=.3,dayF=1,flash=0,spawned=false,showDbg=false,locked=false,invOpen=false,msgs=[],fps=60,swing=0,view=0;
const keys={},mouse={l:false,r:false};
const inv=Array(36).fill(null),armor=[0,0,0,0];let sel=0;
const mobs=[],items=[],parts=[],tnts=[],arrows=[],boats=[];
const signTexts=new Map();
function addInv(id,n){const mx=64;for(let i=0;i<36&&n>0;i++){const s=inv[i];if(s&&s.id==id&&s.n<mx){const a=Math.min(n,mx-s.n);s.n+=a;n-=a}}
 for(let i=0;i<36&&n>0;i++)if(!inv[i]){const a=Math.min(n,mx);inv[i]={id,n:a};n-=a}return n}
for(const i of[I.diamond_pickaxe,I.diamond_sword,I.diamond_axe,B.torch,9,25,19,23,1])addInv(i,64);
const cntT=i=>typeof i=='string'?TAG[i].reduce((a,t)=>a+cntT(t),0):inv.reduce((a,s)=>a+(s&&s.id==i?s.n:0),0);
function takeT(i,n){if(typeof i=='string'){for(const t of TAG[i]){const h=Math.min(n,cntT(t));if(h){takeT(t,h);n-=h}}return}for(let k=35;k>=0&&n>0;k--){const s=inv[k];if(s&&s.id==i){const a=Math.min(n,s.n);s.n-=a;n-=a;if(!s.n)inv[k]=null}}}
const msg=t=>{msgs.push({t:performance.now(),s:t});if(msgs.length>8)msgs.shift();if(typeof chatOpen!='undefined'&&chatOpen)refreshChat()};
const armorPts=()=>armor.reduce((a,i)=>a+(i?ARM[i].pts:0),0);
let SP=null;
for(let r=0;r<900&&!SP;r+=6)for(let a=0;a<(r?8:1)&&!SP;a++){const x=Math.round(Math.cos(a*.785)*r),z=Math.round(Math.sin(a*.785)*r),[h,b]=col(x,z);if(h>SEA+1&&b!=0&&b!=5&&b!=6)SP=[x+.5,h+2,z+.5]}
if(!SP)SP=[.5,SEA+4,.5];
[P.x,P.y,P.z]=SP;
function placeSpawn(){if(!chunkAt(SP[0],SP[2]))return false;
 for(let t=0;t<30;t++){const x=Math.floor(SP[0])+t,z=Math.floor(SP[2]);if(!chunkAt(x,z))continue;
  for(let y=H-1;y>0;y--){const b=getBlock(x,y,z);if(b&&BL[b].s&&!BL[b].leaf&&!(b>=7&&b<=8)){if(!getBlock(x,y+1,z)&&!getBlock(x,y+2,z)){SP=[x+.5,y+1.01,z+.5];[P.x,P.y,P.z]=SP;P.vy=0;spawned=true;return true}break}}}
 return false}
 
/* ================= SKIN (jogador) ================= */
const skinCv=document.createElement('canvas');skinCv.width=skinCv.height=64;const sg=skinCv.getContext('2d');let SKH=64;
function paintBox(u,v,w,h,d,c){sg.fillStyle=c;for(const r of[[u+d,v,w,d],[u+d+w,v,w,d],[u,v+d,d,h],[u+d,v+d,w,h],[u+d+w,v+d,d,h],[u+2*d+w,v+d,w,h]])sg.fillRect(...r)}
function defaultSkin(){skinCv.width=skinCv.height=64;SKH=64;sg.clearRect(0,0,64,64);
 const sk='rgb(224,172,129)',hr='rgb(74,50,30)',sh='rgb(58,110,190)',pn='rgb(48,52,110)',so='rgb(40,40,45)';
 paintBox(0,0,8,8,8,sk);sg.fillStyle=hr;for(const r of[[8,0,8,8],[24,8,8,8],[0,8,8,3],[16,8,8,3],[8,8,8,2]])sg.fillRect(...r);
 sg.fillStyle='#fff';sg.fillRect(9,12,2,1);sg.fillRect(13,12,2,1);sg.fillStyle='#2a4aa0';sg.fillRect(10,12,1,1);sg.fillRect(13,12,1,1);sg.fillStyle='#a05050';sg.fillRect(10,14,4,1);
 paintBox(16,16,8,12,4,sh);
 for(const[u,v]of[[40,16],[32,48]]){paintBox(u,v,4,12,4,sh);sg.fillStyle=sk;for(const r of[[u+4,v+12,4,4],[u+8,v+12,4,4],[u+12,v+12,4,4],[u+16,v+12,4,4]])sg.fillRect(r[0],r[1]+4-4+0,r[2],r[3]);sg.fillRect(u+4,v+4,4,4);sg.fillStyle=sk;sg.fillRect(u,v+4+8,4,4)}
 for(const[u,v]of[[0,16],[16,48]]){paintBox(u,v,4,12,4,pn);sg.fillStyle=so;for(const r of[[u,v+4+9,4,3],[u+4,v+4+9,4,3],[u+8,v+4+9,4,3],[u+12,v+4+9,4,3]])sg.fillRect(...r)}
 upSkin()}
function upSkin(){gl.activeTexture(gl.TEXTURE1);gl.bindTexture(gl.TEXTURE_2D,skinTex);gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,gl.RGBA,gl.UNSIGNED_BYTE,skinCv);gl.activeTexture(gl.TEXTURE0)}
function setSkin(img,save){skinCv.width=img.width;skinCv.height=img.height;sg.clearRect(0,0,img.width,img.height);sg.drawImage(img,0,0);SKH=img.height*64/img.width;upSkin();
 if(save){try{localStorage.setItem('mcjs_skin',skinCv.toDataURL())}catch(e){}}}
defaultSkin();
try{const s=localStorage.getItem('mcjs_skin');if(s){const im=new Image();im.onload=()=>setSkin(im,false);im.src=s}}catch(e){}
// texturas de entidades vindas do pacote do usuário (zombie, skeleton, creeper, steve...)
const ENT={};
function addEntTex(name,img){gl.activeTexture(gl.TEXTURE2);const t=gl.createTexture();gl.bindTexture(gl.TEXTURE_2D,t);gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MIN_FILTER,gl.NEAREST);gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MAG_FILTER,gl.NEAREST);
 gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_WRAP_S,gl.CLAMP_TO_EDGE);gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_WRAP_T,gl.CLAMP_TO_EDGE);gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,gl.RGBA,gl.UNSIGNED_BYTE,img);gl.activeTexture(gl.TEXTURE0);
 ENT[name]={tex:t,th:img.height*64/img.width,legacy:img.height*2<=img.width}}
function bindUnit(u,t){gl.activeTexture(gl.TEXTURE0+u);gl.bindTexture(gl.TEXTURE_2D,t);gl.uniform1i(U.T,u)}
 
/* ================= FÍSICA ================= */
function collides(e){const w=e.w/2;for(let y=Math.floor(e.y);y<=Math.floor(e.y+e.h-1e-3);y++)for(let z=Math.floor(e.z-w);z<=Math.floor(e.z+w-1e-4);z++)for(let x=Math.floor(e.x-w);x<=Math.floor(e.x+w-1e-4);x++){const b=getBlock(x,y,z);if(!b)continue;const q=BL[b];
 if(q.box){if(q.nocol)continue;const mt=getMeta(x,y,z),bx=typeof q.box=='function'?q.box(mt,x,y,z):q.box;for(const o of bx){const hh=q.tall&&o[4]>=.99?1.5:o[4];if(e.x+w>x+o[0]&&e.x-w<x+o[3]&&e.z+w>z+o[2]&&e.z-w<z+o[5]&&e.y+e.h>y+o[1]&&e.y<y+hh)return true}}
 else if(q.s)return true}return false}
function moveEnt(e,dx,dy,dz){
 const n=Math.max(1,Math.ceil(Math.max(Math.abs(dx),Math.abs(dy),Math.abs(dz))/.25));dx/=n;dy/=n;dz/=n;const wasG=e.onGround;e.hitX=false;e.hitZ=false;e.onGround=false;
 for(let i=0;i<n;i++){
  if(dx){e.x+=dx;if(collides(e)){if(wasG&&e.stepUp!==0){e.y+=.6;if(collides(e)){e.y-=.6;e.x-=dx;dx=0;e.hitX=true}}else{e.x-=dx;dx=0;e.hitX=true}}}
  if(dz){e.z+=dz;if(collides(e)){if(wasG&&e.stepUp!==0){e.y+=.6;if(collides(e)){e.y-=.6;e.z-=dz;dz=0;e.hitZ=true}}else{e.z-=dz;dz=0;e.hitZ=true}}}
  if(dy){e.y+=dy;if(collides(e)){e.y-=dy;if(dy<0)e.onGround=true;dy=0;e.vy=0}}}}
const blockAt=(x,y,z)=>getBlock(Math.floor(x),Math.floor(y),Math.floor(z));
function hurtP(n,dx=0,dz=0){if(gm!=0||dead)return;n*=(1-Math.min(20,armorPts())/25)*[0,.6,1,1.4][diff]*(P.fx.resist?.8:1);P.hp-=n;flash=.35;const l=Math.hypot(dx,dz)||1;P.vx=-dx/l*4;P.vz=-dz/l*4;P.vy=Math.max(P.vy,4);if(P.hp<=0)die()}
function die(){dead=true;mouse.l=mouse.r=false;msg('Você morreu! Pressione R para renascer.');
 const dr=(id,n)=>items.push({x:P.x,y:P.y+1,z:P.z,vx:(Math.random()-.5)*4,vy:4,vz:(Math.random()-.5)*4,id,n,age:0,w:.25,h:.25});
 if(!RULES.keepInventory){for(let i=0;i<36;i++)if(inv[i]){dr(inv[i].id,inv[i].n);inv[i]=null}for(let i=0;i<4;i++)if(armor[i]){dr(armor[i],1);armor[i]=0}}}
function respawn(){dead=false;P.hp=20;P.food=20;P.air=10;P.fx={};if(dim!=0)goDim(0,SP[0],SP[2],false);else[P.x,P.y,P.z]=SP;P.vx=P.vy=P.vz=0;P.peak=P.y;if(gm==0)addInv(I.wooden_pickaxe,1)}
function updPlayer(dt){
 if(!chunkAt(P.x,P.z))return;
 if(P.ride){const m=P.ride;if(m.dead||m.dying>0||keys.ShiftLeft){m.rider=false;P.ride=null;P.x=m.x+Math.sin(m.yaw)*.8;P.z=m.z+Math.cos(m.yaw)*.8;P.y=m.y+.25}else{m.rider=true;if(m.boat){m.yaw=P.yaw;const f=(keys.KeyW?1:0)-(keys.KeyS?1:0),r=(keys.KeyD?1:0)-(keys.KeyA?1:0);m.vx+=(-Math.sin(m.yaw)*f+Math.cos(m.yaw)*r)*dt*5;m.vz+=(-Math.cos(m.yaw)*f-Math.sin(m.yaw)*r)*dt*5;m.vx*=Math.pow(.08,dt);m.vz*=Math.pow(.08,dt)}P.x=m.x;P.y=m.y+.55;P.z=m.z;P.vx=P.vy=P.vz=0;P.peak=P.y;return}}
 const fly=P.fly||gm==2;
 const fw=(keys.KeyW?1:0)-(keys.KeyS?1:0),rt=(keys.KeyD?1:0)-(keys.KeyA?1:0),l=Math.hypot(fw,rt)||1;
 P.sprint=(keys.ControlLeft||P.spr)&&fw>0&&(gm!=0||P.food>6);
 let sp=4.3*(P.sprint?1.5:1)*(keys.ShiftLeft&&!fly?.4:1)*(P.fx.speed?1+.2*(P.fx.speed.a||1):1)*(P.speedMul||1);if(fly)sp*=P.sprint?3:2.2;
 const sy=Math.sin(P.yaw),cy=Math.cos(P.yaw);
 let wx=(-sy*fw/l+cy*rt/l)*sp,wz=(-cy*fw/l-sy*rt/l)*sp;
 const inW=[.25,.8,1.35].some(h=>blockAt(P.x,P.y+h,P.z)==10),inL=[.25,.8,1.35].some(h=>blockAt(P.x,P.y+h,P.z)==26);
 if(inW||inL){wx*=.6;wz*=.6}
 const k=Math.min(1,dt*(P.onGround||fly?12:3));P.vx+=(wx-P.vx)*k;P.vz+=(wz-P.vz)*k;
 if(fly)P.vy=((keys.Space?1:0)-(keys.ShiftLeft?1:0))*(P.sprint?12:8);
 else if(ladderAt(P)){P.vy=(keys.Space||fw>0)?3.2:(keys.ShiftLeft?0:-2);P.peak=P.y}
 else if(inW||inL){const surface=blockAt(P.x,P.y+1.55,P.z)==(inW?10:26);P.vy+=(keys.Space?(surface?30:22):-5)*dt;P.vy=Math.max(inL?-2.2:-3.2,Math.min(keys.Space?(surface?6.2:4.8):1.4,P.vy));if(inL)P.vy=Math.max(P.vy,-1.5)}
 else{P.vy-=28*dt;if(P.vy<-50)P.vy=-50;if(keys.Space&&P.onGround)P.vy=8.6*(P.fx.jump?1.3:1);if(P.fx.slowfall)P.vy=Math.max(P.vy,-2)}
 if(gm==2){P.x+=P.vx*dt;P.y+=P.vy*dt;P.z+=P.vz*dt}else{const wasG=P.onGround;let mdx=P.vx*dt,mdz=P.vz*dt;
  if(keys.ShiftLeft&&wasG&&!fly){const supported=(px,pz)=>{const w=P.w*.5-.035,yy=Math.floor(P.y-.06);for(const[ox,oz]of[[-w,-w],[-w,w],[w,-w],[w,w]]){const id=blockAt(px+ox,yy,pz+oz);if(!id||!BL[id].s||BL[id].cross)return false}return true};
   let nx=mdx,nz=mdz;for(let i=0;i<8&&!supported(P.x+nx,P.z+nz);i++){if(Math.abs(nx)>=Math.abs(nz))nx*=.5;else nz*=.5}if(!supported(P.x+nx,P.z+nz)){nx=0;nz=0}mdx=nx;mdz=nz;
  }
  moveEnt(P,mdx,P.vy*dt,mdz);
  if(P.hitX&&P.onGround&&!fly)P.vx=0;
  if(!P.onGround&&!inW&&!fly){if(P.y>P.peak)P.peak=P.y}
  else{if(P.onGround&&!wasG&&!fly&&gm==0&&RULES.fallDamage&&!P.fx.slowfall&&P.peak-P.y>3.4)hurtP(Math.floor(P.peak-P.y-3));P.peak=P.y}
  if(inW||fly)P.peak=P.y}
 const spd=Math.hypot(P.vx,P.vz);P.walkT+=spd*dt*2.2;P.walkAmp+=(Math.min(1,spd/3)*(P.onGround||fly?1:.3)-P.walkAmp)*Math.min(1,dt*10);
 if(P.y<-30)voidFall();
 if(gm==0){
  if(inL&&!P.fx.fireres)hurtP(12*dt);
  if(blockAt(P.x,P.y+1.62,P.z)==10&&!P.fx.waterb){P.air-=dt;if(P.air<=0){P.air=0;P.drown=(P.drown||0)+dt;if(P.drown>1){P.drown=0;hurtP(2)}}}else P.air=Math.min(10,P.air+dt*4);
  P.food=Math.max(0,P.food-dt*(.02+(P.sprint?.12:0)+(mouse.l?.03:0)));
  P.reg=(P.reg||0)+dt;if(P.reg>4){P.reg=0;if(P.food>=18&&P.hp<20)P.hp++;else if(P.food<=0)hurtP(1)}
 }else{P.hp=20;P.food=20;P.air=10}
}
 
/* ================= RAIO / INTERAÇÃO ================= */
function ray(ox,oy,oz,dx,dy,dz,max,solidOnly,liq){
 let x=Math.floor(ox),y=Math.floor(oy),z=Math.floor(oz);const sx=dx>0?1:-1,sy=dy>0?1:-1,sz=dz>0?1:-1;
 const tdx=Math.abs(1/(dx||1e-9)),tdy=Math.abs(1/(dy||1e-9)),tdz=Math.abs(1/(dz||1e-9));
 let tx=(dx>0?x+1-ox:ox-x)*tdx,ty=(dy>0?y+1-oy:oy-y)*tdy,tz=(dz>0?z+1-oz:oz-z)*tdz,t=0;
 const hitBox=(bx,by,bz,box)=>{let lo=0,hi=max,nx=0,ny=0,nz=0;const mn=[bx+box[0],by+box[1],bz+box[2]],mx=[bx+box[3],by+box[4],bz+box[5]],o=[ox,oy,oz],d=[dx,dy,dz];
  for(let a=0;a<3;a++){if(Math.abs(d[a])<1e-10){if(o[a]<mn[a]||o[a]>mx[a])return null;continue}let u=(mn[a]-o[a])/d[a],v=(mx[a]-o[a])/d[a],sign=-Math.sign(d[a]);if(u>v)[u,v]=[v,u];if(u>lo){lo=u;nx=ny=nz=0;if(a==0)nx=sign;else if(a==1)ny=sign;else nz=sign}hi=Math.min(hi,v);if(lo>hi)return null}
  if(hi<0||lo>max)return null;return{t:Math.max(0,lo),nx,ny,nz}};
 for(let i=0;i<96;i++){
  const id=getBlock(x,y,z),b=id&&BL[id];if(id&&(liq||!b.liq)&&(!solidOnly||b.s)){
   const mt=getMeta(x,y,z),boxes=b.box?(typeof b.box=='function'?b.box(mt,x,y,z):b.box):[[0,0,0,1,1,1]];let best=null;
   for(const q of boxes||[]){const h=hitBox(x,y,z,q);if(h&&(!best||h.t<best.t))best=h}
   if(best&&best.t<=max)return{x,y,z,id,nx:best.nx,ny:best.ny,nz:best.nz,t:best.t};
  }
  if(tx<ty&&tx<tz){x+=sx;t=tx;tx+=tdx}else if(ty<tz){y+=sy;t=ty;ty+=tdy}else{z+=sz;t=tz;tz+=tdz}
  if(t>max)break}
 return null}
function rayMob(ox,oy,oz,dx,dy,dz,max){let best=null;
 for(const m of mobs){if(m.dying>0)continue;const w=m.w/2,mn=[m.x-w,m.y,m.z-w],mx=[m.x+w,m.y+m.h,m.z+w],o=[ox,oy,oz],d=[dx,dy,dz];let t0=0,t1=max,ok=true;
  for(let i=0;i<3;i++){if(Math.abs(d[i])<1e-9){if(o[i]<mn[i]||o[i]>mx[i]){ok=false;break}}else{let a=(mn[i]-o[i])/d[i],b=(mx[i]-o[i])/d[i];if(a>b)[a,b]=[b,a];t0=Math.max(t0,a);t1=Math.min(t1,b);if(t0>t1){ok=false;break}}}
  if(ok&&(!best||t0<best.t))best={m,t:t0}}
 return best}
const toolOf=id=>IT[id]&&IT[id].tool?IT[id]:null;
function mineInfo(b,held){const tl=toolOf(held),kind=b.mat=='r'?'p':b.mat=='w'?'a':'s';let sp=1;if(tl&&tl.tool==kind)sp=tl.spd;
 const ok=b.need===undefined||b.need<0||(tl&&tl.tool=='p'&&tl.tier>=b.need);return{t:Math.max(.05,b.hard*(ok?1.5:5)/sp),ok}}
function spawnItem(x,y,z,id,n){items.push({x,y,z,vx:(Math.random()-.5)*2,vy:3,vz:(Math.random()-.5)*2,id,n,age:0,w:.25,h:.25})}
function burst(x,y,z,tl,n,col){for(let i=0;i<n;i++)parts.push({x,y,z,vx:(Math.random()-.5)*4,vy:Math.random()*4,vz:(Math.random()-.5)*4,life:.5+Math.random()*.5,tl,col})}
function ignite(x,y,z,t=3){edit(x,y,z,0);tnts.push({x:x+.5,y:y+.5,z:z+.5,t,vy:2,w:.98,h:.98})}
function explode(x,y,z,r,pw,nb){
 for(let dx=-r;dx<=r;dx++)for(let dy=-r;dy<=r;dy++)for(let dz=-r;dz<=r;dz++){
  if(nb)continue;const q=Math.hypot(dx,dy,dz);if(q>r+Math.random()*.6-.3)continue;
  const bx=Math.floor(x)+dx,by=Math.floor(y)+dy,bz=Math.floor(z)+dz,id=getBlock(bx,by,bz);
  if(!id||id==11||id==24||BL[id].liq)continue;
  if(id==23){setBlock(bx,by,bz,0);tnts.push({x:bx+.5,y:by+.5,z:bz+.5,t:.4+Math.random()*.8,vy:0,w:.98,h:.98});continue}
  if(Math.random()<.12&&gm==0&&!BL[id].leaf){const dd=BL[id].drop!==undefined?BL[id].drop:id;if(dd)spawnItem(bx+.5,by+.5,bz+.5,dd,1)}
  setBlock(bx,by,bz,0)}
 burst(x,y,z,TN.white,30,[1,.6,.1]);
 for(const m of mobs){const d=Math.hypot(m.x-x,m.y+.5-y,m.z-z);if(d<r*2)hurtMob(m,(1-d/(r*2))*pw,m.x-x,m.z-z,true)}
 const d=Math.hypot(P.x-x,P.y+1-y,P.z-z);if(d<r*2){hurtP((1-d/(r*2))*pw,x-P.x,z-P.z);if(gm!=2)P.vy+=6*(1-d/(r*2))}}
function attack(mb){const s=inv[sel],it=s&&IT[s.id];let dmg=(it&&it.dmg?it.dmg:1)+(P.fx.strength?3:0);if(gm==1)dmg=100;hurtMob(mb,dmg,mb.x-P.x,mb.z-P.z,false,true)}
function equip(id){const a=ARM[id].armor,old=armor[a];armor[a]=id;return old}
/* ================= MOBS (modelos, IA, animações) ================= */
const W1=[1,1,1];
function human(skin,shirt,pants,face,th){const a=th?.125:.25,bx=th?.3125:.375,lx=th?.125:.125;return[
 {s:[.5,.5,.5],o:[0,1.75,0],c:skin,f:TN[face],pv:-.25,hd:1,uv:[0,0,8,8,8]},{s:[.5,.75,.25],o:[0,1.125,0],c:shirt,uv:[16,16,8,12,4]},
 {s:[a,.75,a],o:[-bx,1.125,0],c:th?shirt:skin,sw:.15,ph:1,ang:-1.4,uv:[40,16,th?2:4,12,th?2:4]},{s:[a,.75,a],o:[bx,1.125,0],c:th?shirt:skin,sw:.15,ph:-1,ang:-1.4,uv:[32,48,th?2:4,12,th?2:4],uvL:[40,16,th?2:4,12,th?2:4]},
 {s:[a,.75,a],o:[-lx,.375,0],c:pants,sw:.8,ph:1,uv:[0,16,th?2:4,12,th?2:4]},{s:[a,.75,a],o:[lx,.375,0],c:pants,sw:.8,ph:-1,uv:[16,48,th?2:4,12,th?2:4],uvL:[0,16,th?2:4,12,th?2:4]}]}
function quad(c1,c2,c3,L,Wd,Hb,lh,face,hs_=.45){const g=[[1,1],[-1,-1],[1,-1],[-1,1]].map((q,i)=>({s:[.2,lh,.2],o:[q[0]*(Wd/2-.1),lh/2,q[1]*(L/2-.15)],c:c3,sw:.7,ph:i<2?1:-1}));
 return[{s:[Wd,Hb,L],o:[0,lh+Hb/2,0],c:c1},{s:[hs_,hs_,hs_],o:[0,lh+Hb*.7,L/2+.2],c:c2,f:TN[face],pv:-hs_/2,hd:1},...g]}
const MT={
 pig:{hp:10,w:.8,h:.9,sp:1.6,kind:'p',parts:quad([1,.75,.8],W1,[.95,.7,.75],.85,.55,.5,.35,'pface'),drops:[[I.porkchop,1,2]]},
 cow:{hp:10,w:.9,h:1.4,sp:1.4,kind:'p',parts:quad([.38,.24,.14],[.33,.22,.14],[.3,.2,.12],1.1,.7,.7,.6,'face'),drops:[[I.beef,1,2],[I.leather,0,2]]},
 sheep:{hp:8,w:.9,h:1.3,sp:1.4,kind:'p',parts:quad([.95,.95,.95],[.4,.35,.32],[.75,.65,.55],.95,.65,.6,.45,'face'),drops:[[I.mutton,1,2],[B.wool_white,1,1]]},
 chicken:{hp:4,w:.4,h:.7,sp:1.3,kind:'p',flap:1,parts:[{s:[.3,.3,.45],o:[0,.4,0],c:[.95,.95,.95]},{s:[.2,.25,.2],o:[0,.62,.27],c:[.95,.95,.95],pv:-.12,hd:1},{s:[.1,.06,.12],o:[0,.58,.4],c:[.95,.6,.1]},{s:[.05,.1,.05],o:[0,.5,.4],c:[.8,.1,.1]},
  {s:[.06,.25,.06],o:[-.08,.125,0],c:[.9,.7,.15],sw:.9,ph:1},{s:[.06,.25,.06],o:[.08,.125,0],c:[.9,.7,.15],sw:.9,ph:-1},
  {s:[.05,.2,.3],o:[-.19,.45,0],c:[.9,.9,.9],wing:1},{s:[.05,.2,.3],o:[.19,.45,0],c:[.9,.9,.9],wing:-1}],drops:[[I.chicken,1,1],[I.feather,0,2]]},
 villager:{hp:20,w:.6,h:1.95,sp:1.3,kind:'v',parts:[{s:[.5,.65,.5],o:[0,1.7,0],c:[.9,.7,.6],f:TN.face,pv:-.3,hd:1},{s:[.12,.22,.12],o:[0,1.65,.3],c:[.75,.55,.45]},{s:[.55,.9,.3],o:[0,1.05,0],c:[.45,.3,.2]},
  {s:[.65,.22,.22],o:[0,1.15,.18],c:[.45,.3,.2]},{s:[.22,.6,.25],o:[-.12,.3,0],c:[.35,.25,.15],sw:.8,ph:1},{s:[.22,.6,.25],o:[.12,.3,0],c:[.35,.25,.15],sw:.8,ph:-1}],drops:[[I.emerald,0,1],[I.bread,0,1]]},
 wolf:{hp:8,w:.6,h:.85,sp:1.9,kind:'n',range:20,dmg:3,parts:[...quad([.8,.8,.8],[.85,.85,.85],[.75,.75,.75],.7,.35,.4,.4,'face',.35),{s:[.1,.1,.35],o:[0,.72,-.45],c:[.78,.78,.78],tail:1}],drops:[[I.bone,0,1]]},
 zombie:{hp:20,w:.6,h:1.95,sp:2.3,kind:'h',range:30,dmg:3,burn:1,ent:'zombie',parts:human([.55,.75,.45],[.2,.35,.6],[.25,.25,.55],'zface'),drops:[[I.rotten,0,2],[I.carrot,0,1],[I.potato,0,1],[I.iron_ingot,0,1]]},
 skeleton:{hp:20,w:.6,h:1.95,sp:2.1,kind:'h',range:30,burn:1,ent:'skeleton',parts:human([.82,.82,.78],[.75,.75,.72],[.7,.7,.66],'sface',1),drops:[[I.bone,0,2],[I.arrow,0,2]]},
 bomber:{hp:20,w:.6,h:1.7,sp:2.2,kind:'h',range:30,ent:'creeper',parts:[{s:[.5,.5,.5],o:[0,1.375,0],c:W1,f:TN.cface,pv:-.25,hd:1,uv:[0,0,8,8,8]},{s:[.5,.75,.25],o:[0,.75,0],c:[.3,.7,.3],uv:[16,16,8,12,4]},
  ...[[-1,1],[1,1],[-1,-1],[1,-1]].map((q,i)=>({s:[.25,.375,.25],o:[q[0]*.125,.19,q[1]*.2],c:[.25,.6,.25],sw:.7,ph:i%2?1:-1,uv:[0,16,4,6,4]}))],drops:[[I.gunpowder,0,2]]},
 spider:{hp:16,w:1,h:.7,sp:3,kind:'h',range:25,dmg:2,parts:[{s:[.7,.4,.5],o:[0,.45,-.1],c:[.25,.2,.18]},{s:[.4,.35,.4],o:[0,.45,.35],c:W1,f:TN.spface,pv:-.17,hd:1},
  ...[0,1,2,3].flatMap(i=>[-1,1].map(sg_=>({s:[.8,.07,.07],o:[sg_*.55,.38,-.3+i*.22],c:[.2,.16,.14],sw:.3,ph:(i%2?1:-1)*sg_})))],drops:[[I.string,0,2],[I.spider_eye,0,1]]},
 stalker:{hp:40,w:.6,h:2.8,sp:3.2,kind:'n',range:30,dmg:6,tp:1,parts:[{s:[.5,.5,.5],o:[0,2.0,0],c:[.5,.5,.5],f:TN.eface,pv:-.25,hd:1},{s:[.5,.75,.25],o:[0,1.375,0],c:[.1,.07,.13]},
  {s:[.15,1,.15],o:[-.325,1.25,0],c:[.1,.07,.13],sw:.15,ph:1},{s:[.15,1,.15],o:[.325,1.25,0],c:[.1,.07,.13],sw:.15,ph:-1},{s:[.2,1,.2],o:[-.1,.5,0],c:[.1,.07,.13],sw:.8,ph:1},{s:[.2,1,.2],o:[.1,.5,0],c:[.1,.07,.13],sw:.8,ph:-1}],drops:[[I.ender_pearl,0,1]]},
 slime:{hp:4,w:.8,h:.8,sp:3,kind:'h',range:20,dmg:2,slime:1,parts:[],drops:[[I.slime,0,2]]}};
for(const k in MT)MT[k].host=MT[k].kind=='h';
function slimeParts(m){const z=m.size*.5+.3,sq=m.squish||0;return[{s:[z*(1-sq*.2),z*(1+sq*.35),z*(1-sq*.2)],o:[0,z*(1+sq*.35)/2,0],c:MT[m.t].col||W1,f:TN.slface,all:undefined,pv:0}]}
function spawnMob(t,x,y,z,size){const T=MT[t];if(!T)return null;const m={t,x,y,z,vx:0,vy:0,vz:0,w:T.w,h:T.h,yaw:Math.random()*6.28,hp:T.hp,sw:0,st:0,tm:0,walk:0,hurt:0,cd:1,fuse:0,flee:0,fire:0,kx:0,kz:0,shoot:2,ang:0,hy:0,roll:0,dying:0,hop:0,squish:0,flapT:0,tpT:5};
 if(T.slime){m.size=size||(1+Math.floor(Math.random()*3));m.hp=[0,4,8,16][m.size];m.w=m.h=m.size*.5+.3}mobs.push(m);return m}
function groundY(x,z){if(!chunkAt(x,z))return -1;for(let y=H-1;y>0;y--){const b=getBlock(Math.floor(x),y,Math.floor(z));if(b&&(BL[b].s||BL[b].liq))return BL[b].liq?-1:y+1}return -1}
function hurtMob(m,dmg,dx,dz,expl,byP){if(m.dying>0)return;m.hp-=dmg;m.hurt=.35;const l=Math.hypot(dx,dz)||1;m.kx=dx/l*(expl?9:5);m.kz=dz/l*(expl?9:5);m.vy=expl?7:4.5;m.tm=0;
 const T=MT[m.t];if(T.kind=='p'||T.kind=='v'){m.flee=4;m.yaw=Math.atan2(dx,dz)}else if(byP&&(T.kind=='n'||m.t=='spider'))m.ang=25;
 if(T.tp&&byP&&Math.random()<.5)tpMob(m,10)}
function tpMob(m,r){for(let i=0;i<12;i++){const a=Math.random()*6.283,x=m.x+Math.cos(a)*r,z=m.z+Math.sin(a)*r,y=groundY(x,z);if(y>0){burst(m.x,m.y+1,m.z,TN.white,12,[.7,.2,.9]);m.x=x;m.y=y;m.z=z;burst(x,y+1,z,TN.white,12,[.7,.2,.9]);return}}}
function finishMob(m){const T=MT[m.t];
 if(!m.noDrop)for(const d of T.drops){const n=d[1]+Math.floor(Math.random()*(d[2]-d[1]+1));if(n&&(d[2]-d[1]<2||Math.random()<.9))spawnItem(m.x,m.y+.5,m.z,d[0],n)}
 if(T.slime&&m.size>1)for(let i=0;i<2+(Math.random()<.5?1:0);i++){const c=spawnMob('slime',m.x+(Math.random()-.5),m.y+.3,m.z+(Math.random()-.5),m.size-1);if(c)c.vy=5}
 burst(m.x,m.y+.7,m.z,TN.white,10,[.8,.1,.1]);m.dead=true;if(!m.noDrop)addXp(T.host?5:2)}
function updMob(m,dt){
 if(!chunkAt(m.x,m.z))return;
 const T=MT[m.t],K=T.kind;
 if(m.dying>0){m.dying-=dt;m.roll=Math.min(1.57,(1-m.dying/.6)*1.57);if(m.dying<=0)finishMob(m);m.vy-=26*dt;moveEnt(m,0,m.vy*dt,0);return}
 const dx=P.x-m.x,dz=P.z-m.z,dist=Math.hypot(dx,dz);let sp=0;
 m.hurt=Math.max(0,m.hurt-dt);m.cd-=dt;m.tm-=dt;m.flee=Math.max(0,m.flee-dt);m.ang=Math.max(0,m.ang-dt);m.hop-=dt;
 const canT=gm==0&&!dead,hostileNow=T.host&&!peace&&!(m.t=='spider'&&dayF>.5&&m.ang<=0);
 const aggro=canT&&dist<(T.range||0)&&(hostileNow||m.ang>0);
 if(m===P.ride){m.yaw=P.yaw+Math.PI;sp=((keys.KeyW?1:0)-(keys.KeyS?.5:0))*T.sp*2.4;if(keys.Space&&m.onGround)m.vy=9}
 else if(aggro){m.yaw=Math.atan2(dx,dz);sp=T.sp;
  if(m.t=='skeleton'||m.t=='wisp'){if(dist<5)sp=-T.sp;else if(dist<11)sp=0;m.shoot-=dt;
   if(dist<18&&m.shoot<=0){m.shoot=2;const ex=m.x,ey=m.y+1.5,ez=m.z,tx=P.x-ex,ty=P.y+1.2-ey,tz=P.z-ez,l=Math.hypot(tx,ty,tz);arrows.push({fire:m.t=='wisp',x:ex,y:ey,z:ez,vx:tx/l*22,vy:ty/l*22+dist*.18,vz:tz/l*22,life:4})}}
  else if(m.t=='bomber'){if(dist<3){m.fuse+=dt;sp=0}else m.fuse=Math.max(0,m.fuse-dt*2);if(m.fuse>1.5){explode(m.x,m.y+.5,m.z,3,14,!RULES.mobGriefing);m.noDrop=true;m.dead=true;return}}
  else if(T.slime){const dm=Math.max(2,m.size);if(m.onGround){sp=0;if(m.hop<=0){m.vy=6.5+m.size*.4;m.hop=.9+Math.random()*.5;m.hopV=T.sp*.8}}else sp=m.hopV||0;
   if(dist<m.w+.6&&Math.abs(P.y-m.y)<1.6&&m.cd<=0){m.cd=1;hurtP(dm,dx,dz)}}
  else if(m.t=='stalker'){m.tpT-=dt;if(m.tpT<=0){m.tpT=3+Math.random()*3;if(dist>5)tpMob(m,3)}if(dist<2&&m.cd<=0){m.cd=1;hurtP(T.dmg,dx,dz)}}
  else if(dist<1.4&&Math.abs(P.y-m.y)<1.6&&m.cd<=0){m.cd=1;hurtP(T.dmg||3,dx,dz)}
 }else{m.fuse=Math.max(0,m.fuse-dt);
  if(m.tm<=0){m.tm=1.5+Math.random()*3;m.walk=Math.random()<.55?1:0;m.yaw=Math.random()*6.283}
  sp=m.flee>0?T.sp*2:m.walk?T.sp*.7:0;
  if(K=='v'&&m.flee<=0){let zb=null,zd=9;for(const o of mobs)if(o.t=='zombie'){const d=Math.hypot(o.x-m.x,o.z-m.z);if(d<zd){zd=d;zb=o}}if(zb){m.yaw=Math.atan2(m.x-zb.x,m.z-zb.z);m.flee=1;}}
  if(T.slime&&m.onGround){sp=0;if(m.hop<=0){m.hop=1.2+Math.random()*1.5;m.vy=5.5;m.hopV=T.sp*.5;m.yaw=Math.random()*6.28}}else if(T.slime)sp=m.hopV||0;
  if(T.tp&&Math.random()<dt*.05)tpMob(m,8)}
 // olhar para o jogador (cabeça)
 const want=(dist<7&&!aggro&&m.flee<=0)?Math.max(-.9,Math.min(.9,((Math.atan2(dx,dz)-m.yaw+9.4248)%6.2832)-3.1416)):(aggro?0:0);
 m.hy+=(want-m.hy)*Math.min(1,dt*6);
 if(T.swim&&blockAt(m.x+Math.sin(m.yaw)*.9,m.y+.2,m.z+Math.cos(m.yaw)*.9)!=10)m.yaw+=2.2+Math.random();
 const fx=Math.sin(m.yaw)*sp+m.kx,fz=Math.cos(m.yaw)*sp+m.kz;m.kx*=Math.pow(.02,dt);m.kz*=Math.pow(.02,dt);
 const inW=blockAt(m.x,m.y+.3,m.z)==10;
 if(T.swim)m.vy=inW?Math.sin(m.st*.7+m.x)*.5:Math.max(m.vy-26*dt,-40);else if(T.fly){m.fy=m.fy||0;m.vy+=(m.fy-m.vy)*Math.min(1,dt*3)}else if(inW)m.vy=Math.min(1.8,m.vy+30*dt);else{m.vy=Math.max(-40,m.vy-26*dt);if(T.flap&&m.vy<-2){m.vy=-2;m.flapT=.2}}
 if(T.fly){if(aggro)m.fy=Math.max(-3,Math.min(3,(P.y+2.5-m.y)*1.5));else if(Math.random()<dt*.8)m.fy=(Math.random()-.5)*4}
 moveEnt(m,fx*dt,m.vy*dt,fz*dt);
 if((m.hitX||m.hitZ)&&(m.onGround||m.t=='spider')&&sp)m.vy=m.t=='spider'?5:8;
 m.flapT=Math.max(0,m.flapT-dt);if(T.flapAlways)m.flapT=.2;
 const spd=Math.hypot(fx,fz);m.st+=spd*dt*2.5;m.sw+=(Math.min(1,spd/2)-m.sw)*Math.min(1,dt*8);
 if(T.slime)m.squish+=((m.onGround?.35:-.3)-m.squish)*Math.min(1,dt*10);
 if(T.burn&&dayF>.6&&!inW){const c=chunkAt(m.x,m.z),yy=c.hm[(Math.floor(m.x)&15)+(Math.floor(m.z)&15)*16];if(m.y>=yy){m.hp-=dt*2;if(Math.random()<.2)burst(m.x,m.y+1,m.z,TN.white,1,[1,.5,0])}}
 if(m.t=='stalker'&&inW){m.hp-=dt*4;tpMob(m,8)}
 if(m.y<-10||dist>80){m.dead=true;m.noDrop=true;return}
 if(m.hp<=0){m.dying=.6;m.fuse=0}}
function drawBox(ox,oy,oz,yaw,pt,ang,tn,lt,roll,hy,uv,thU,grow){
 grow=grow||1;const sx=pt.s[0]*grow,sy=pt.s[1]*grow,sz=pt.s[2]*grow,px=pt.o[0],py=pt.o[1],pz=pt.o[2],pv=pt.pv!==undefined?pt.pv:sy/2;
 const cy=Math.cos(yaw),sy_=Math.sin(yaw),ca=Math.cos(ang),sa=Math.sin(ang),ch=Math.cos(hy||0),sh_=Math.sin(hy||0),cr=Math.cos(roll||0),sr=Math.sin(roll||0);
 let R=null;if(uv){const[u,v,w,h,d]=uv;R=[[u+d+w,v+d,d,h],[u,v+d,d,h],[u+d,v,w,d],[u+d+w,v,w,d],[u+d,v+d,w,h],[u+2*d+w,v+d,w,h]]}
 for(let f=0;f<6;f++){
  const own=pt.all!=null||(f==4&&pt.f!=null),tl=pt.all!=null?pt.all:(f==4&&pt.f!=null?pt.f:TN.white),cl=pt.c||W1,s=SHD[f];
  for(let k=0;k<4;k++){const cc=FC[f][k];let lx=(cc[0]-.5)*sx,ly=(cc[1]-.5)*sy,lz=(cc[2]-.5)*sz;
   if(ang){const yy=ly-pv;ly=yy*ca-lz*sa+pv;lz=yy*sa+lz*ca}
   if(pt.hd&&hy){const a=lx*ch+lz*sh_;lz=-lx*sh_+lz*ch;lx=a}
   let x1=px+lx,y1=py+ly;const z1=pz+lz;
   if(roll){const a=x1*cr-y1*sr;y1=x1*sr+y1*cr;x1=a}
   let u_,v_;if(R){const r=R[f];u_=(r[0]+UVC[k][0]*r[2])/64;v_=(r[1]+UVC[k][1]*r[3])/thU}else{u_=tu(tl,UVC[k][0]);v_=tv(tl,UVC[k][1])}
   const c0=uv?1:cl[0],c1=uv?1:cl[1],c2=uv?1:cl[2];
   DV.push(ox+x1*cy+z1*sy_,oy+y1,oz-x1*sy_+z1*cy,u_,v_,c0*s*tn[0],c1*s*tn[1],c2*s*tn[2],lt[0],lt[1])}
  DI.push(DN,DN+1,DN+2,DN,DN+2,DN+3);DN+=4}}
const LT=[1,.3];
function mobTint(m){return m.hurt>0?[1.5,.5,.5]:(m.fuse>0&&Math.floor(m.fuse*8)%2?[3,3,3]:(m.ang>0&&MT[m.t].kind=='n'?[1.2,.8,.8]:W1))}
function mobAng(m,pt){let a=(pt.ang||0)+(pt.sw?Math.sin(m.st)*pt.sw*m.sw*pt.ph:0);
 if(pt.wing)a=0;if(pt.tail)a=Math.sin(m.st*.5)*.2;return a}
function drawMobs(){
 const skinned={};
 for(const m of mobs){const T=MT[m.t],tn=mobTint(m),en=T.ent&&ENT[T.ent];
  if(T.slime){for(const pt of slimeParts(m))drawBox(m.x,m.y,m.z,m.yaw,pt,0,tn,LT,m.roll,0);continue}
  if(en){(skinned[T.ent]=skinned[T.ent]||[]).push(m);continue}
  for(const pt of T.parts){let ang=mobAng(m,pt);if(T.flap&&pt.wing)ang=0;
   if(pt.wing&&m.flapT>0){drawBox(m.x,m.y,m.z,m.yaw,{...pt,o:[pt.o[0]+pt.wing*.08,pt.o[1]+.05,pt.o[2]]},0,tn,LT,m.roll+pt.wing*.5*Math.sin(performance.now()*.03),0)}
   else drawBox(m.x,m.y,m.z,m.yaw,pt,ang,tn,LT,m.roll,m.hy)}}
 dflush();
 for(const n in skinned){const en=ENT[n];bindUnit(2,en.tex);
  for(const m of skinned[n]){const T=MT[m.t],tn=mobTint(m);for(const pt of T.parts){const uv=en.legacy&&pt.uvL?pt.uvL:pt.uv;drawBox(m.x,m.y,m.z,m.yaw,pt,mobAng(m,pt),tn,LT,m.roll,m.hy,uv,en.th)}}
  dflush();bindUnit(0,tex)}}
const id2t=id=>id<IB?(BL[id].t[0]==TN.gt?TN.gs:BL[id].t[2]):IT[id].tile;
function drawPlayerModel(){
 const yaw=P.yaw+Math.PI,w=P.walkAmp,st=P.walkT,tn=W1,lt=[1,.45],th=SKH,leg=SKH<=32;
 const ar=Math.sin(st)*.9*w,mining=mouse.l&&locked;
 const rA=mining?-1.1+Math.sin(swing)*.7:-ar,lA=ar;
 const x=P.x,y=P.y,z=P.z,pit=-P.pitch;
 const body=[
  [{s:[.5,.5,.5],o:[0,1.75,0],pv:-.25,hd:1},pit,[0,0,8,8,8],0],
  [{s:[.5,.75,.25],o:[0,1.125,0]},0,[16,16,8,12,4],0],
  [{s:[.25,.75,.25],o:[-.375,1.125,0]},rA,[40,16,4,12,4],0],
  [{s:[.25,.75,.25],o:[.375,1.125,0]},lA,leg?[40,16,4,12,4]:[32,48,4,12,4],0],
  [{s:[.25,.75,.25],o:[-.125,.375,0]},ar,[0,16,4,12,4],0],
  [{s:[.25,.75,.25],o:[.125,.375,0]},-ar,leg?[0,16,4,12,4]:[16,48,4,12,4],0]];
 bindUnit(1,skinTex);
 for(const[pt,a,uv]of body)drawBox(x,y,z,yaw,pt,a,tn,lt,0,0,uv,th);
 const ov=[[0,[32,0,8,8,8],1.12],...(leg?[]:[[1,[16,32,8,12,4],1.06],[2,[40,32,4,12,4],1.06],[3,[48,48,4,12,4],1.06],[4,[0,32,4,12,4],1.06],[5,[0,48,4,12,4],1.06]])];
 for(const[i,uv,g]of ov){const[pt,a]=body[i];drawBox(x,y,z,yaw,pt,a,tn,lt,0,0,uv,th,g)}
 dflush();bindUnit(0,tex);
 // armadura sobre o modelo
 const A=(i)=>armor[i]?ARM[armor[i]].ac:null,g=1.14;
 let c=A(0);if(c)drawBox(x,y,z,yaw,{s:[.5*g,.5*g,.5*g],o:[0,1.75,0],pv:-.25,hd:1,c},pit,[1,1,1],lt,0,0);
 c=A(1);if(c){drawBox(x,y,z,yaw,{s:[.5*g,.75*g,.25*g],o:[0,1.125,0],c},0,W1,lt,0,0);
  drawBox(x,y,z,yaw,{s:[.3,.45,.3],o:[-.375,1.275,0],c},rA,W1,lt,0,0);drawBox(x,y,z,yaw,{s:[.3,.45,.3],o:[.375,1.275,0],c},lA,W1,lt,0,0)}
 c=A(2);if(c){drawBox(x,y,z,yaw,{s:[.5*g,.2,.25*g],o:[0,.8,0],c},0,W1,lt,0,0);drawBox(x,y,z,yaw,{s:[.3,.5,.3],o:[-.125,.5,0],c},ar,W1,lt,0,0);drawBox(x,y,z,yaw,{s:[.3,.5,.3],o:[.125,.5,0],c},-ar,W1,lt,0,0)}
 if(armor[3]){c=A(3);drawBox(x,y,z,yaw,{s:[.31,.3,.31],o:[-.125,.15,0],pv:.6,c},ar,W1,lt,0,0);drawBox(x,y,z,yaw,{s:[.31,.3,.31],o:[.125,.15,0],pv:.6,c},-ar,W1,lt,0,0)}
 dflush()}
function drawEntities(){
 drawMobs();
 for(const b of boats){const c=WOODS[b.wood]?.[2]||[190,160,90],yaw=b.yaw;drawBox(b.x,b.y,b.z,yaw,{s:[1.25,.22,.72],o:[0,.05,0],c:c.map(v=>v/255)},0,W1,[1,.5]);drawBox(b.x,b.y+.12,b.z,yaw,{s:[1.05,.12,.5],o:[0,0,0],c:c.map(v=>Math.max(0,v-35)/255)},0,W1,[1,.5]);}

 for(const it of items){drawBox(it.x,it.y+.15+Math.sin(it.age*3)*.05,it.z,it.age*1.5,{s:[.3,.3,.3],o:[0,0,0],all:id2t(it.id),c:it.id<IB&&BL[it.id].tint?(BL[it.id].tc||[.45,.75,.3]):W1},0,W1,[1,.4])}
 for(const t of tnts){const f=Math.floor(t.t*5)%2?[3,3,3]:W1,sc=1+(t.t<.5?.1:0);drawBox(t.x,t.y,t.z,0,{s:[sc,sc,sc],o:[0,0,0],all:TN.tnts},0,f,[1,.5])}
 for(const p of parts)drawBox(p.x,p.y,p.z,0,{s:[.12,.12,.12],o:[0,0,0],all:p.tl,c:p.col},0,W1,[1,.5]);
 for(const a of arrows)drawBox(a.x,a.y,a.z,Math.atan2(a.vx,a.vz),{s:a.fire?[.2,.2,.2]:[.08,.08,.6],o:[0,0,0],c:a.fire?[1,.5,.1]:[.6,.45,.25]},0,W1,[1,.5]);
 dflush();
 if(view>0&&gm!=2&&!dead)drawPlayerModel()}
let spawnT=1,initMobs=false;
function updEntities(dt){
 for(let i=boats.length-1;i>=0;i--){const b=boats[i];b.age+=dt;const water=blockAt(b.x,b.y-.05,b.z)==10||blockAt(b.x,b.y-.35,b.z)==10,nearLava=[-.1,.2,.5].some(h=>blockAt(b.x,b.y+h,b.z)==26);if(nearLava){if(P.ride===b){P.ride=null;b.rider=false;P.y=b.y+1}boats.splice(i,1);msg('O barco foi destruído pela lava');continue}if(water){b.y+=(Math.floor(b.y)+.88-b.y)*Math.min(1,dt*5);b.vx*=Math.pow(.2,dt);b.vz*=Math.pow(.2,dt)}else{b.vx*=Math.pow(.03,dt);b.vz*=Math.pow(.03,dt);b.y-=dt*2}const nx=b.x+b.vx*dt,nz=b.z+b.vz*dt;if(!collides({x:nx,y:b.y,w:1.05,h:.4,z:nz})){b.x=nx;b.z=nz}else{b.vx=b.vz=0}if(!chunkAt(b.x,b.z)||b.age>1800)boats.splice(i,1)}
 for(const m of mobs)updMob(m,dt);for(let i=mobs.length-1;i>=0;i--)if(mobs[i].dead)mobs.splice(i,1);
 for(let i=items.length-1;i>=0;i--){const it=items[i];it.age+=dt;it.vy-=22*dt;if(chunkAt(it.x,it.z)){moveEnt(it,it.vx*dt,it.vy*dt,it.vz*dt);if(it.onGround){it.vx*=.8;it.vz*=.8}}
  if(!dead&&it.age>.6&&Math.hypot(it.x-P.x,it.y-P.y-.8,it.z-P.z)<1.6&&gm!=2){const left=addInv(it.id,it.n);if(left<it.n){it.n=left;if(!left)items.splice(i,1)}continue}
  if(it.age>300)items.splice(i,1)}
 for(let i=parts.length-1;i>=0;i--){const p=parts[i];p.vy-=14*dt;p.x+=p.vx*dt;p.y+=p.vy*dt;p.z+=p.vz*dt;p.life-=dt;if(p.life<=0)parts.splice(i,1)}
 for(let i=tnts.length-1;i>=0;i--){const t=tnts[i];t.t-=dt;t.vy-=20*dt;if(chunkAt(t.x,t.z)){const q={x:t.x,y:t.y-.5,z:t.z,w:.9,h:.9,vy:t.vy};moveEnt(q,0,t.vy*dt,0);t.y=q.y+.5;t.vy=q.vy}
  if(t.t<=0){explode(t.x,t.y,t.z,4,18);tnts.splice(i,1)}}
 for(let i=arrows.length-1;i>=0;i--){const a=arrows[i];a.vy-=9*dt;a.x+=a.vx*dt;a.y+=a.vy*dt;a.z+=a.vz*dt;a.life-=dt;
  const b=blockAt(a.x,a.y,a.z);let rm=a.life<=0||(b&&BL[b].s);
  if(!rm&&!dead&&gm==0&&Math.abs(a.x-P.x)<.4&&Math.abs(a.z-P.z)<.4&&a.y>P.y&&a.y<P.y+1.8){hurtP(3,a.vx,a.vz);rm=true}
  if(rm)arrows.splice(i,1)}
 spawnTick(dt);
}
 
/* ================= SISTEMAS EXTRAS ================= */
const DN_=['Overworld','Nether','End'];
const N6=[[1,0,0],[-1,0,0],[0,1,0],[0,-1,0],[0,0,1],[0,0,-1]],H4=[[1,0],[-1,0],[0,1],[0,-1]];
const kk=(x,y,z)=>x+','+y+','+z;
const getMeta=(x,y,z)=>{if(y<0||y>=H)return 0;const c=chunks.get((x>>4)+','+(z>>4));return c?c.m[(x&15)+16*((z&15)+16*y)]:0};
let pend=null,diff=2,xp=0,weather=0,wT=200,lightning=0,lt_=10,flT=0,rsT=0,tkT=0,animT=0,SENS=.0022,FOVB=70;
const RULES={keepInventory:0,doDaylightCycle:1,mobGriefing:1,doMobSpawning:1,doWeatherCycle:1,fallDamage:1};
let RSS=DIMS[0].rss,FLQ=DIMS[0].flq,TKS=DIMS[0].tks;
const RSTIME=new Map(),rsLast=new Map();
function rsReg(x,y,z,id){const k=kk(x,y,z),b=BL[id];if(b&&b.rs)RSS.add(k);else RSS.delete(k);if(b&&b.tick)TKS.add(k);else TKS.delete(k)}
function fluidNote(x,y,z){for(const[dx,dy,dz]of[[0,0,0],...N6]){const b=getBlock(x+dx,y+dy,z+dz);if(b==10||b==26)FLQ.add(kk(x+dx,y+dy,z+dz))}}
function fluidInto(x,y,z,id,meta){const opp=id==10?26:10;
 for(const[dx,dy,dz]of N6){const b=getBlock(x+dx,y+dy,z+dz);if(b==opp){if(id==10){const mm=getMeta(x+dx,y+dy,z+dz);setBlock(x+dx,y+dy,z+dz,mm==0?24:4)}else{setBlock(x,y,z,4);return}}}
 setBlock(x,y,z,id,meta)}
const FLCD=new Map();
function fluidTick(dt){
 flT+=dt;if(flT<.25)return;flT=0;if(!FLQ.size)return;
 const list=[...FLQ].slice(0,450);for(const k of list)FLQ.delete(k);
 for(const k of list){const[x,y,z]=k.split(',').map(Number);if(!chunkAt(x,z))continue;const id=getBlock(x,y,z);if(id!=10&&id!=26)continue;
  const lava=id==26,now=performance.now()/1000,wait=lava?(dim==1?.5:1.5):.25,last=FLCD.get(k)||-999;
  if(now-last<wait){FLQ.add(k);continue}FLCD.set(k,now);
  const m=getMeta(x,y,z),maxL=lava?(dim==1?7:3):7;
  if(m!=0&&m!=8){let supported=false;
   if(getBlock(x,y+1,z)==id)supported=true;
   else for(const[dx,dz]of H4){if(getBlock(x+dx,y,z+dz)==id){const nm=getMeta(x+dx,y,z+dz);if(nm==8||nm<m){supported=true;break}}}
   if(!supported){setBlock(x,y,z,0);continue}}
  const lvl=m==8?0:m,below=getBlock(x,y-1,z);
  if(!below||BL[below].cross){fluidInto(x,y-1,z,id,8);continue}
  if(below==id)continue;
  // Fontes de água podem se regenerar quando ficam entre duas fontes e têm apoio sólido.
  if(!lava&&m>0&&m<8){let sources=0;for(const[dx,dz]of H4)if(getBlock(x+dx,y,z+dz)==10&&getMeta(x+dx,y,z+dz)==0)sources++;if(sources>=2&&below&&BL[below].s){setBlock(x,y,z,10,0);continue}}
  if(lvl<maxL){for(const[dx,dz]of H4){const nx=x+dx,nz=z+dz,n=getBlock(nx,y,nz);if(!n||BL[n].cross){
    // Não espalha horizontalmente se houver uma queda imediatamente acessível por outro lado.
    let next=lvl+1;for(const[ax,az]of H4){const q=getBlock(nx+ax,y-1,nz+az);if((!q||BL[q].cross)&&getBlock(nx+ax,y-1,nz+az)!=id){next=Math.max(1,lvl);break}}
    if(next<=maxL)fluidInto(nx,y,nz,id,next)
   }}}
 }
}
function settle(x,y,z){const id=getBlock(x,y,z);if(!id||!BL[id].fall)return;let yy=y;while(yy>0&&!getBlock(x,yy-1,z))yy--;if(yy!=y){setBlock(x,y,z,0);setBlock(x,yy,z,id)}}
function settleUp(x,y,z){for(let k=1;k<=8;k++){const b=getBlock(x,y+k,z);if(b&&BL[b].fall)settle(x,y+k,z);else break}}
/* ---------- redstone ---------- */
function powerAt(x,y,z){for(const[dx,dy,dz]of N6){const b=getBlock(x+dx,y+dy,z+dz);if(!b)continue;const R=BL[b].rs;if(!R)continue;const mt=getMeta(x+dx,y+dy,z+dz);
 if(R=='wire'&&(mt&15)>0)return true;if((R=='lever'||R=='button'||R=='plate')&&(mt&8))return true;if(R=='block')return true;if(R=='torch_on'&&dy!=1)return true}return false}
function entOn(x,y,z){const h=(e,w)=>Math.abs(e.x-x-.5)<.5+w&&Math.abs(e.z-z-.5)<.5+w&&e.y>=y-.2&&e.y<y+.6;
 if(gm!=2&&h(P,.2))return true;for(const m of mobs)if(h(m,m.w/2))return true;for(const it of items)if(h(it,.1))return true;return false}
const IMM=id=>BL[id].hard>=1e8||id==24||BL[id].cont||id==B.piston||id==B.pistonhead;
function pistonExtend(x,y,z,n){const d=N6[n];let cx=x+d[0],cy=y+d[1],cz=z+d[2];const chain=[];
 for(;;){const b=getBlock(cx,cy,cz);if(!b||BL[b].liq||BL[b].cross)break;if(IMM(b)||chain.length>=12)return false;chain.push([cx,cy,cz,b,getMeta(cx,cy,cz)]);cx+=d[0];cy+=d[1];cz+=d[2]}
 for(let i=chain.length-1;i>=0;i--){const c=chain[i];setBlock(c[0]+d[0],c[1]+d[1],c[2]+d[2],c[3],c[4])}
 setBlock(x+d[0],y+d[1],z+d[2],B.pistonhead,n);return true}
function rsTick(){
 const now=performance.now(),wires=[];
 for(const k of RSS){const[x,y,z]=k.split(',').map(Number),b=getBlock(x,y,z),R=b&&BL[b].rs;if(!R){RSS.delete(k);continue}if(R=='wire')wires.push([x,y,z])}
 const pw=new Map(),q=[];
 for(const[x,y,z]of wires){let p=0;for(const[dx,dy,dz]of N6){const b=getBlock(x+dx,y+dy,z+dz);if(!b)continue;const R=BL[b].rs;if(!R||R=='wire')continue;const mt=getMeta(x+dx,y+dy,z+dz);
  if(((R=='lever'||R=='button'||R=='plate')&&(mt&8))||R=='block'||R=='torch_on')p=15}
  if(p){pw.set(kk(x,y,z),15);q.push([x,y,z,15])}}
 const NW=[[1,0,0],[-1,0,0],[0,0,1],[0,0,-1],[1,1,0],[-1,1,0],[0,1,1],[0,1,-1],[1,-1,0],[-1,-1,0],[0,-1,1],[0,-1,-1]];
 for(let qi=0;qi<q.length;qi++){const[x,y,z,p]=q[qi];if(p<=1)continue;for(const[dx,dy,dz]of NW){const nx=x+dx,ny=y+dy,nz=z+dz,nb=getBlock(nx,ny,nz);
  if(nb&&BL[nb].rs=='wire'){const kn=kk(nx,ny,nz);if((pw.get(kn)||0)<p-1){pw.set(kn,p-1);q.push([nx,ny,nz,p-1])}}}}
 for(const[x,y,z]of wires){const m=pw.get(kk(x,y,z))||0;if(getMeta(x,y,z)!==m)setBlock(x,y,z,B.rs_wire,m)}
 for(const k of[...RSS]){const[x,y,z]=k.split(',').map(Number),id=getBlock(x,y,z),R=id&&BL[id].rs,mt=getMeta(x,y,z);if(!R)continue;
  if(R=='lamp'){const want=powerAt(x,y,z)?B.lamp_on:B.lamp;if(id!=want)setBlock(x,y,z,want)}
  else if(R=='torch_on'||R=='torch_off'){const want=powerAt(x,y-1,z)?B.rtorch_off:B.rtorch;if(id!=want)setBlock(x,y,z,want)}
  else if(R=='button'){if((mt&8)&&now>(RSTIME.get(k)||0))setBlock(x,y,z,id,mt&~8)}
  else if(R=='plate'){const on=entOn(x,y,z);if(on!=!!(mt&8))setBlock(x,y,z,id,on?mt|8:mt&~8)}
  else if(R=='piston'){const p=powerAt(x,y,z),n=mt&7,ext=mt&8;
   if(p&&!ext){if(pistonExtend(x,y,z,n))setBlock(x,y,z,B.piston,n|8)}
   else if(!p&&ext){const d=N6[n];if(getBlock(x+d[0],y+d[1],z+d[2])==B.pistonhead)setBlock(x+d[0],y+d[1],z+d[2],0);setBlock(x,y,z,B.piston,n)}}
  else if(R=='use'){const b=BL[id];if(b.door&&(mt&8))continue;const p=b.door?(powerAt(x,y,z)||powerAt(x,y+1,z)):powerAt(x,y,z),last=rsLast.get(k)||false;
   if(p!=last){rsLast.set(k,p);if(id==23){if(p)ignite(x,y,z)}else if(b.door){const lm=(mt&~4)|(p?4:0);setBlock(x,y,z,id,lm);setBlock(x,y+1,z,id,lm|8)}else if(b.trap)setBlock(x,y,z,id,(mt&~4)|(p?4:0))}}}}
function toggleDoor(x,y,z){const mt=getMeta(x,y,z),id=getBlock(x,y,z);if(BL[id].iron)return;const ly=(mt&8)?y-1:y,mm=getMeta(x,ly,z)^4;edit(x,ly,z,getBlock(x,ly,z),mm);edit(x,ly+1,z,getBlock(x,ly+1,z),mm|8)}
/* ---------- contêineres e fornalha ---------- */
const SMELT={[I.beef]:[I.cooked_beef,1],[I.porkchop]:[I.cooked_porkchop,1],[I.chicken]:[I.cooked_chicken,1],[I.mutton]:[I.cooked_mutton,1],[I.cod]:[I.cooked_cod,1],[I.salmon]:[I.cooked_salmon,1],[I.potato]:[I.baked_potato,1],
 5:[19,1],4:[3,1],[I.clay_ball]:[I.brick,1],7:[I.charcoal,1],[B.birch_log]:[I.charcoal,1],[B.clay]:[B.terracotta,1],[B.netherrack]:[I.brick,1]};
const FUEL={[I.coal]:80,[I.charcoal]:80,9:15,7:15,[I.stick]:5,[I.blaze_rod]:120,[I.lava_bucket]:1000,[B.coal_block]:800};
for(const w in WOODS)if(w!='crimson'&&w!='warped'){for(const k of ['planks','log','wood','stripped_log','stripped_wood'])FUEL[B[w+'_'+k]]=15}
function getCont(x,y,z,t){const k=kk(x,y,z);let c=conts.get(k);if(!c){c=t=='furnace'?{t,s:[null,null,null],burn:0,bm:1,prog:0,x,y,z}:{t,s:Array(27).fill(null),x,y,z};conts.set(k,c)}return c}
function openCont(c){UI.cont=c;invOpen=true;UI.held=null;UI.dragStart=null;document.exitPointerLock()}
function openChest(x,y,z){
 const a=getCont(x,y,z,'chest'),mt=getMeta(x,y,z),alongZ=(mt&3)<2;
 const dirs=alongZ?[[0,1],[0,-1]]:[[1,0],[-1,0]];
 for(const[dx,dz]of dirs){const nx=x+dx,nz=z+dz;
  if(getBlock(nx,y,nz)==B.chest&&(getMeta(nx,y,nz)&3)==(mt&3)){
   const b=getCont(nx,y,nz,'chest');openCont({t:'doubleChest',a,b});return;
  }
 }
 openCont(a);
}
function furnaceTick(dt){for(const c of conts.values()){if(c.t!='furnace')continue;const inp=c.s[0],r=inp&&SMELT[inp.id],out=c.s[2],can=r&&(!out||(out.id==r[0]&&out.n+r[1]<=64));
 if(c.burn<=0&&can&&c.s[1]&&FUEL[c.s[1].id]){c.bm=c.burn=FUEL[c.s[1].id];c.s[1].n--;if(!c.s[1].n)c.s[1]=null}
 if(c.burn>0){c.burn-=dt;if(can){c.prog+=dt;if(c.prog>=10){c.prog=0;inp.n--;if(!inp.n)c.s[0]=null;if(out)out.n+=r[1];else c.s[2]={id:r[0],n:r[1]};xp+=1}}else c.prog=Math.max(0,c.prog-dt*2)}else c.prog=0;
 const lit=c.burn>0;if(c.lit!==lit){c.lit=lit;const b=getBlock(c.x,c.y,c.z);if(b==B.furnace||b==B.furnace_on)setBlock(c.x,c.y,c.z,lit?B.furnace_on:B.furnace)}}}
/* ---------- dimensões ---------- */
function genDim(d,bio,cx,cz){
 let top=0;const I3=(x,y,z)=>x+16*(z+16*y);
 if(dim==1){
  for(let z=0;z<16;z++)for(let x=0;x<16;x++){const wx=cx*16+x,wz=cz*16+z;const nb=hs(wx>>5,wz>>5,901);bio[x+z*16]=nb<.2?57:nb<.4?58:nb<.62?59:nb<.82?60:56;
   for(let y=0;y<80;y++){let id=0;
    if(y==0||y==79)id=11;
    else if(y<=2||y>=77)id=hs(wx,y,wz)<(y<=2?(3-y)*.3:(y-76)*.3)?11:B.netherrack;
    else{const base=n3(wx/30,y/18,wz/30,81)*.65+n3(wx/11,y/9,wz/11,83)*.35,edge=Math.min(y,79-y)/14,dens=base+(1-Math.min(1,edge))*.32;if(dens>.55)id=B.netherrack}
    if(!id&&y<=18)id=26;
    d[I3(x,y,z)]=id;if(id&&y>top)top=y}}
  for(let z=0;z<16;z++)for(let x=0;x<16;x++){const wx=cx*16+x,wz=cz*16+z;
   for(let y=3;y<77;y++){const i=I3(x,y,z);if(d[i]!=B.netherrack)continue;const up=d[I3(x,y+1,z)];
    if(up==0){if(y>19&&n2(wx/9,wz/9,7)>.68)d[i]=B.soul_sand;else if(hs(wx,y,wz^9)<.05)d[i]=6}
    else if(up==26&&y<=20)d[i]=B.magma;
    else if(hs(wx,y,wz^77)<.01)d[i]=B.quartz_ore}}
  for(let z=0;z<16;z++)for(let x=0;x<16;x++){const wx=cx*16+x,wz=cz*16+z;
   for(let y=8;y<76;y++){if(d[I3(x,y,z)]==0&&d[I3(x,y+1,z)]==B.netherrack&&hs(wx,y,wz^33)<.006){
    for(let dx=-1;dx<=1;dx++)for(let dy=-1;dy<=0;dy++)for(let dz=-1;dz<=1;dz++){const a=x+dx,b=y+dy,c=z+dz;if(a>=0&&a<16&&c>=0&&c<16&&d[I3(a,b,c)]==0)d[I3(a,b,c)]=25}}}}
  return 79}
 for(let z=0;z<16;z++)for(let x=0;x<16;x++){const wx=cx*16+x,wz=cz*16+z,dist=Math.hypot(wx,wz);bio[x+z*16]=dist<42?61:(dist>260&&dist<420?65:(dist>90&&dist<260?(n2(wx/38,wz/38,97)>.78?64:63):(dist>42&&dist<90?62:64)));let topY=-1,bot=0;
  if(dist<42){topY=64+Math.round(n2(wx/12,wz/12,95)*2);bot=64-Math.round((42-dist)*.45)-Math.round(n2(wx/9,wz/9,96)*4)}
  else if(dist>90&&dist<420){const q=n2(wx/38,wz/38,97);if(q>(dist>260?.82:.72)){topY=(dist>260?46:52)+Math.round(n2(wx/8,wz/8,98)*3);bot=topY-Math.round((q-(dist>260?.82:.72))*48)-2}}
  if(topY>=0)for(let y=Math.max(1,bot);y<=Math.min(topY,78);y++){d[I3(x,y,z)]=B.end_stone;if(y>top)top=y}
  for(let p=0;p<10;p++){const a=p/10*6.2832,px=Math.round(Math.cos(a)*32),pz=Math.round(Math.sin(a)*32),ht=70+((p*5)%9);
   if(Math.hypot(wx-px,wz-pz)<=2.6)for(let y=60;y<=ht;y++){d[I3(x,y,z)]=24;if(y>top)top=y}}}
 return top}
const endTop=(wx,wz)=>Math.hypot(wx,wz)<42?64+Math.round(n2(wx/12,wz/12,95)*2):64;
function goDim(n,x,z,viaPortal){
 for(const c of chunks.values()){freeMesh(c.mo);freeMesh(c.mt);c.mo=c.mt=null;c.dirty=true}
 dim=n;chunks=DIMS[n].chunks;mods=DIMS[n].mods;conts=DIMS[n].conts;RSS=DIMS[n].rss;FLQ=DIMS[n].flq;TKS=DIMS[n].tks;
 mobs.length=0;items.length=0;boats.length=0;arrows.length=0;tnts.length=0;parts.length=0;
 P.x=x;P.z=z;P.y=n==0?70:H-2;P.vx=P.vy=P.vz=0;P.mine=null;P.ride=null;P.fly=gm==2;spawned=false;initMobs=true;pend={dim:n,x,z,portal:viaPortal};msg('Dimensão: '+DN_[n])}
function buildPortal(x,y,z){for(let i=-1;i<=2;i++)for(let h=-1;h<=3;h++){const edge=i==-1||i==2||h==-1||h==3;setBlock(x+i,y+h,z,edge?24:B.portal,0)}
 for(let dz=-1;dz<=1;dz+=2)for(let i=-1;i<=2;i++){setBlock(x+i,y-1,z+dz,24);for(let h=0;h<3;h++)setBlock(x+i,y+h,z+dz,0)}}
function resolvePend(){const p=pend;if(!chunkAt(p.x,p.z))return;const X=Math.floor(p.x),Z=Math.floor(p.z);let y=-1;
 if(p.dim==0){y=groundY(p.x,p.z);if(y<0)y=SEA+2}
 else if(p.dim==1){for(let yy=76;yy>20;yy--)if(!getBlock(X,yy,Z)&&!getBlock(X,yy+1,Z)&&getBlock(X,yy-1,Z)&&BL[getBlock(X,yy-1,Z)].s){y=yy;break}
  if(y<0){y=40;for(let i=-2;i<=2;i++)for(let k=-2;k<=2;k++){setBlock(X+i,39,Z+k,B.netherrack);for(let h=0;h<4;h++)setBlock(X+i,40+h,Z+k,0)}}}
 else{y=endTop(X,Z)+1;for(let i=-2;i<=2;i++)for(let k=-2;k<=2;k++){setBlock(X+i,y-1,Z+k,24);setBlock(X+i,y,Z+k,0);setBlock(X+i,y+1,Z+k,0)}}
 let px=X+.5,pz=Z+.5;
 if(p.portal){let fp=null;for(let dx=-8;dx<=8&&!fp;dx++)for(let dz=-8;dz<=8&&!fp;dz++)for(let yy=2;yy<H-2;yy++)if(getBlock(X+dx,yy,Z+dz)==B.portal){fp=[X+dx,yy,Z+dz];break}
  if(fp){let yy=fp[1];while(getBlock(fp[0],yy-1,fp[2])==B.portal)yy--;px=fp[0]+.5;pz=fp[2]+1.6;y=yy}else{buildPortal(X,y,Z);pz=Z+1.6}}
 P.x=px;P.y=y+.01;P.z=pz;P.vy=0;P.peak=P.y;P.portalCd=5;spawned=true;pend=null}
function tryPortal(x,y,z){for(const ax of[0,1]){const dx=ax==0?1:0,dz=ax==0?0:1;
 let l=0;while(l<10&&getBlock(x-dx*(l+1),y,z-dz*(l+1))==0&&getBlock(x-dx*(l+1),y-1,z-dz*(l+1))==24)l++;
 let r=0;while(r<10&&getBlock(x+dx*(r+1),y,z+dz*(r+1))==0&&getBlock(x+dx*(r+1),y-1,z+dz*(r+1))==24)r++;
 const w=l+r+1;if(w<2||w>10)continue;const x0=x-dx*l,z0=z-dz*l;
 if(getBlock(x0-dx,y,z0-dz)!=24||getBlock(x0+dx*w,y,z0+dz*w)!=24)continue;
 let h=0;while(h<21){let ok=true;for(let i=0;i<w;i++)if(getBlock(x0+dx*i,y+h,z0+dz*i)!=0){ok=false;break}if(!ok)break;if(getBlock(x0-dx,y+h,z0-dz)!=24||getBlock(x0+dx*w,y+h,z0+dz*w)!=24)break;h++}
 if(h<3)continue;let top=true;for(let i=0;i<w;i++)if(getBlock(x0+dx*i,y+h,z0+dz*i)!=24){top=false;break}if(!top)continue;
 for(let hh=0;hh<h;hh++)for(let i=0;i<w;i++)setBlock(x0+dx*i,y+hh,z0+dz*i,B.portal,ax);return true}return false}
function voidFall(){if(dim==2){P.x=.5;P.z=.5;P.y=endTop(0,0)+3;P.vy=0;P.peak=P.y;hurtP(6)}else{P.y=60;P.vy=0;if(gm==0)hurtP(10)}}
const ladderAt=e=>{for(let y=Math.floor(e.y);y<=Math.floor(e.y+1);y++){const b=getBlock(Math.floor(e.x),y,Math.floor(e.z));if(b&&BL[b].climb)return true}return false};
/* ---------- efeitos, xp, clima, animação ---------- */
function effectsTick(dt){const f=P.fx;for(const k in f){f[k].t-=dt;if(f[k].t<=0){delete f[k];continue}
 if(k=='regen'&&gm==0){f[k].c=(f[k].c||0)+dt;if(f[k].c>1.5){f[k].c=0;P.hp=Math.min(20,P.hp+1)}}
 if(k=='poison'&&gm==0&&P.hp>1){f[k].c=(f[k].c||0)+dt;if(f[k].c>1.2){f[k].c=0;P.hp-=1;flash=.2}}
 if(k=='hunger'&&gm==0)P.food=Math.max(0,P.food-dt*.3);
 if(k=='lev')P.vy=Math.max(P.vy,2.5)}}
const addXp=n=>{xp+=n};
function growTree(x,y,z,sap=B.sapling){const kind=BL[sap]?.saplingType||'oak',th=(kind=='spruce'||kind=='poplar'?6:4)+(Math.random()*2|0),log=B[kind+'_log'],leaf=kind=='poplar'?[B.poplar_red_leaves,B.poplar_orange_leaves,B.poplar_yellow_leaves][Math.random()*3|0]:B[kind+'_leaves'];for(let k=1;k<=th;k++){const b=getBlock(x,y+k,z);if(b&&!BL[b].cross)return}
 for(let k=0;k<=th;k++)setBlock(x,y+k,z,log);
 for(let dy=th-2;dy<=th+1;dy++){const rad=dy>=th?1:2;for(let dz=-rad;dz<=rad;dz++)for(let dx=-rad;dx<=rad;dx++){if(Math.abs(dx)==rad&&Math.abs(dz)==rad&&Math.random()<.5)continue;if(dx==0&&dz==0&&dy<=th)continue;if(!getBlock(x+dx,y+dy,z+dz))setBlock(x+dx,y+dy,z+dz,leaf)}}}
function randTick(){let n=0;for(const k of TKS){if(++n>400)break;const[x,y,z]=k.split(',').map(Number),id=getBlock(x,y,z),b=BL[id];if(!b||!b.tick){TKS.delete(k);continue}
 const mt=getMeta(x,y,z);
 if(id==B.wheat){if(mt<7&&Math.random()<.25)setBlock(x,y,z,id,mt+1)}
 else if(b.saplingType){if(Math.random()<.06)growTree(x,y,z,id)}
 else if(id==B.farmland){let wet=weather>0;for(let dx=-4;dx<=4&&!wet;dx++)for(let dz=-4;dz<=4;dz++)if(getBlock(x+dx,y,z+dz)==10||getBlock(x+dx,y+1,z+dz)==10){wet=true;break}
  const crop=getBlock(x,y+1,z)==B.wheat;if(wet!=!!(mt&1))setBlock(x,y,z,id,wet?1:0);if(!wet&&!crop&&Math.random()<.1)setBlock(x,y,z,2)}}}
const PK=new Set(),ANIM={},FLUID_SRC={};
function animProc(nm,t){const i=TN[nm],ox=(i%TPR)*16,oy=(i/TPR|0)*16,im=ac.createImageData(16,16);
 for(let y=0;y<16;y++)for(let x=0;x<16;x++){const o=(y*16+x)*4,d=im.data;
  if(nm=='water'){const a=x*.78+y*1.12-t*3.2,b=x*.34-y*.62+t*1.7,r=Math.sin(a+Math.sin(b)*1.4)+Math.cos(b+a*.23),foam=Math.max(0,r-.92);d[o]=12+foam*24;d[o+1]=58+foam*80+(r+2)*7;d[o+2]=125+foam*85+(r+2)*9;d[o+3]=foam>0?175:135}
  else if(nm=='lava'){const v=Math.sin(x*.55+t*.7+Math.sin(y*.4-t*.4)*2)+Math.cos(y*.62-t*.55+Math.sin(x*.3+t*.25));const hot=Math.max(0,Math.min(1,(v+1.2)/2.4));d[o]=160+hot*95;d[o+1]=28+hot*145;d[o+2]=4+hot*35;d[o+3]=255}
  else{const v=Math.sin(x*.7+t*2+Math.sin(y*.5+t))*.5+Math.sin(y*.8-t*1.5+Math.cos(x*.4))*.5,w=v*.5+.5;d[o]=90+w*70;d[o+1]=20+w*30;d[o+2]=170+w*60;d[o+3]=200}}
 ac.putImageData(im,ox,oy)}
function animStatic(nm,t){const src=FLUID_SRC[nm];if(!src)return animProc(nm,t);const i=TN[nm],ox=(i%TPR)*16,oy=(i/TPR|0)*16,out=ac.createImageData(16,16),d=out.data,a=src.data,shift=Math.floor(t*(nm=='water'?3:1));
 for(let y=0;y<16;y++)for(let x=0;x<16;x++){const sx=(x+shift+(nm=='water'?Math.floor(y*.35):0))&15,sy=(y+Math.floor(shift*(nm=='water'?.65:.35)))&15,si=(sy*16+sx)*4,di=(y*16+x)*4;d[di]=a[si];d[di+1]=a[si+1];d[di+2]=a[si+2];d[di+3]=a[si+3]}
 ac.putImageData(out,ox,oy)}
function animTick(dt){animT+=dt;if(animT<.2)return;animT=0;const t=performance.now()/1000;
 for(const nm of['water','lava','portal']){const a=ANIM[nm];if(a)paintTile(nm,a.bm,null,Math.floor(t*(nm=='water'?6:3))%a.n,false);else if(nm=='water'||nm=='lava')animStatic(nm,t*(nm=='lava'?.5:1));else if(!PK.has(nm))animProc(nm,t)}
 uploadAtlas();gl.activeTexture(gl.TEXTURE0)}
function worldTick(dt){
 effectsTick(dt);fluidTick(dt);furnaceTick(dt);animTick(dt);
 rsT+=dt;if(rsT>.1){rsT=0;if(RSS.size)rsTick()}
 tkT+=dt;if(tkT>1){tkT=0;randTick()}
 if(dim==0&&RULES.doWeatherCycle){wT-=dt;if(wT<=0){const r=Math.random();weather=r<.55?0:r<.85?1:2;wT=100+Math.random()*400;}
  if(weather==2){lt_-=dt;if(lt_<=0){lt_=6+Math.random()*14;lightning=.7}}}else if(dim)weather=0;
 lightning=Math.max(0,lightning-dt*1.5);
 P.portalCd=Math.max(0,(P.portalCd||0)-dt);
 if(!dead&&gm!=2&&(blockAt(P.x,P.y+.5,P.z)==B.portal||blockAt(P.x,P.y+1.5,P.z)==B.portal)){P.portalT=(P.portalT||0)+dt;
  if(P.portalT>(gm==1?.3:2)&&P.portalCd<=0){P.portalT=0;P.portalCd=5;if(dim==0)goDim(1,P.x/8,P.z/8,true);else if(dim==1)goDim(0,P.x*8,P.z*8,true)}}else P.portalT=0}
function hudExtra(g,hg){
 if(gm==0){const lv=Math.floor(Math.sqrt(xp/4)),pr=Math.max(0,Math.min(1,(xp-4*lv*lv)/(4*(2*lv+1))));if(JAVA_GUI.xpBg){g.drawImage(JAVA_GUI.xpBg,hg.x0,hg.y0-6,hg.w,5);if(JAVA_GUI.xpFill&&pr>0)g.drawImage(JAVA_GUI.xpFill,0,0,Math.max(1,JAVA_GUI.xpFill.width*pr),JAVA_GUI.xpFill.height,hg.x0,hg.y0-6,hg.w*pr,5)}else{g.fillStyle='rgba(0,0,0,.6)';g.fillRect(hg.x0,hg.y0-6,hg.w,4);g.fillStyle='#7f3';g.fillRect(hg.x0,hg.y0-6,hg.w*pr,4)}if(lv>0)txt(''+lv,W/2,hg.y0-9,14,'#7f3','center')}
 let y=40;for(const k in P.fx){txt(`${k} ${Math.ceil(P.fx[k].t)}s`,W-8,y,13,'#fc8','right');y+=16}
 txt(['SOBREVIVÊNCIA','CRIATIVO','ESPECTADOR'][gm]+' · '+DN_[dim]+(view?' · 3ª pessoa':'')+(weather?(weather==2?' · tempestade':' · chuva'):''),8,HH-8,13,'#ccc')}
function drawWeather(g){if(!weather||dim)return;g.strokeStyle='rgba(170,190,255,.5)';g.lineWidth=1;g.beginPath();for(let i=0;i<150;i++){const x=Math.random()*W,y=Math.random()*HH;g.moveTo(x,y);g.lineTo(x-3,y+16)}g.stroke();
 if(lightning>0){g.fillStyle=`rgba(255,255,255,${Math.min(.6,lightning)})`;g.fillRect(0,0,W,HH)}}
/* ---------- uso de itens / quebra de blocos ---------- */
const fdir=()=>{const fx=-Math.sin(P.yaw),fz=-Math.cos(P.yaw);return Math.abs(fx)>Math.abs(fz)?(fx>0?0:1):(fz>0?2:3)};
const nIdx=(nx,ny,nz)=>nx==1?0:nx==-1?1:ny==1?2:ny==-1?3:nz==1?4:5;
function sleepBed(t){if(dim!=0){msg('Você não pode dormir aqui');return}SP=[t.x+.5,t.y+1.01,t.z+.5];if(dayF<.4||weather==2){tod=.3;weather=0;msg('Você dormiu. Bom dia!')}else msg('Ponto de renascimento definido (só dá pra dormir à noite)')}
function absorbWater(x,y,z){const q=N6.map(([dx,dy,dz])=>[x+dx,y+dy,z+dz,1]),seen=new Set(),limit=64;let count=0;while(q.length&&count<limit){const[a,b,c,d]=q.shift(),k=kk(a,b,c);if(seen.has(k)||d>6)continue;seen.add(k);if(getBlock(a,b,c)!=10)continue;setBlock(a,b,c,0);count++;for(const[dx,dy,dz]of N6)q.push([a+dx,b+dy,c+dz,d+1])}if(count){msg('A esponja absorveu '+count+' blocos de água');return true}return false}
function useItem(){
 if(gm==2)return;
 const s=inv[sel],t=P.target,it=s&&IT[s.id],ex=P.x,ey=P.y+1.62,ez=P.z,cp=Math.cos(P.pitch),d=[-Math.sin(P.yaw)*cp,Math.sin(P.pitch),-Math.cos(P.yaw)*cp];
 const consume=()=>{if(gm==0){s.n--;if(!s.n)inv[sel]=null}};
 const mh=rayMob(ex,ey,ez,d[0],d[1],d[2],3.5);
 if(mh&&mh.m.t=='horse'&&(!t||mh.t<t.t)){P.ride=mh.m;msg('Montado! (Shift desmonta)');return}
 if(t&&!keys.ShiftLeft){const id=t.id,b=BL[id],mt=getMeta(t.x,t.y,t.z);
  if(b.strip&&it&&it.tool=='a'){edit(t.x,t.y,t.z,b.strip,mt);return}
  if(b.sign){const k=kk(t.x,t.y,t.z),old=signTexts.get(k)||['','','',''];const v=prompt('Texto da placa (use | para separar até 4 linhas)',old.join(' | '));if(v!==null)signTexts.set(k,v.split('|').slice(0,4).map(x=>x.trim()));return}
  if(b.cont){if(b.cont=='chest')openChest(t.x,t.y,t.z);else openCont(getCont(t.x,t.y,t.z,b.cont));return}
  if(b.door){toggleDoor(t.x,t.y,t.z);return}
  if(b.gate){edit(t.x,t.y,t.z,id,mt^4);return}
  if(b.trap){if(!b.iron)edit(t.x,t.y,t.z,id,mt^4);return}
  if(b.rs=='lever'){edit(t.x,t.y,t.z,id,mt^8);return}
  if(b.rs=='button'){if(!(mt&8)){edit(t.x,t.y,t.z,id,mt|8);RSTIME.set(kk(t.x,t.y,t.z),performance.now()+1000)}return}
  if(b.bed){sleepBed(t);return}
  if(id==22){invOpen=true;UI.cont={t:'crafting',s:UI.craft3};UI.held=null;UI.dragStart=null;document.exitPointerLock();return}}
 if(!s)return;
 if(it&&it.armor!==undefined){const old=equip(s.id);s.n--;if(!s.n)inv[sel]=null;if(old)addInv(old,1);return}
 if(it&&it.food&&(P.food<20||gm!=0)){if(gm==0){P.food=Math.min(20,P.food+it.food);if(it.heal)P.fx.regen={t:it.heal,a:1};s.n--;if(!s.n)inv[sel]=null}burst(P.x,P.y+1.4,P.z,it.tile,6);return}
 if(it&&it.boat){const r=ray(ex,ey,ez,d[0],d[1],d[2],6,false,true);if(!r||r.id!=10){msg('Barcos só podem ser colocados na água');return}const bx=r.x+.5,bz=r.z+.5,by=r.y+.05;if(blockAt(bx,by+.25,bz)==26||blockAt(bx,by+.8,bz)==26){msg('Barcos não funcionam na lava');return}const boat={boat:1,wood:it.boat,x:bx,y:by,z:bz,vx:0,vz:0,yaw:P.yaw,dead:false,rider:false,w:1.35,h:.45,age:0};boats.push(boat);if(gm==0)consume();return}
 if(it&&it.key=='bucket'){const r=ray(ex,ey,ez,d[0],d[1],d[2],5,false,true);if(r&&(r.id==10||r.id==26)&&getMeta(r.x,r.y,r.z)==0){setBlock(r.x,r.y,r.z,0);if(gm==0){s.n--;if(!s.n)inv[sel]=null;addInv(r.id==10?I.water_bucket:I.lava_bucket,1)}}return}
 if(it&&(it.key=='water_bucket'||it.key=='lava_bucket')){if(t){fluidInto(t.x+t.nx,t.y+t.ny,t.z+t.nz,it.key=='water_bucket'?10:26,0);if(gm==0){inv[sel]={id:I.bucket,n:1}}}return}
 if(it&&it.key=='flint_steel'){if(t){if(!tryPortal(t.x+t.nx,t.y+t.ny,t.z+t.nz)&&t.id==23)ignite(t.x,t.y,t.z)}return}
 if(it&&it.tool=='h'){if(t&&(t.id==1||t.id==2)&&!getBlock(t.x,t.y+1,t.z))edit(t.x,t.y,t.z,B.farmland,0);return}
 if(it&&it.key=='seeds'){if(t&&t.id==B.farmland&&!getBlock(t.x,t.y+1,t.z)){edit(t.x,t.y+1,t.z,B.wheat,0);consume()}return}
 if(it&&it.door){if(!t)return;const x=t.x+t.nx,y=t.y+t.ny,z=t.z+t.nz;if(getBlock(x,y,z)||getBlock(x,y+1,z))return;const bid=B['door_'+it.door],f=fdir();edit(x,y,z,bid,f);edit(x,y+1,z,bid,f|8);consume();return}
 if(!t||s.id>=IB)return;
 const b=BL[s.id],x=t.x+t.nx,y=t.y+t.ny,z=t.z+t.nz,cur=getBlock(x,y,z);
 if(b.slab&&t.id==s.id){const tm=getMeta(t.x,t.y,t.z);if((t.ny==1&&tm==0)||(t.ny==-1&&tm==1)){edit(t.x,t.y,t.z,b.slab,0);consume();return}}
 if(cur&&!BL[cur].liq&&!BL[cur].cross)return;
 const hy=ey+d[1]*t.t,fy=hy-Math.floor(hy),upper=t.ny==-1||(t.ny==0&&fy>.5);let meta=0;
 if(b.slab)meta=upper?1:0;else if(b.stairs)meta=fdir()|(upper?4:0);else if(b.gate)meta=fdir()<2?1:0;else if(b.trap)meta=fdir()|(upper?8:0);else if(b.log)meta=t.ny!=0?0:t.nx!=0?1:2;else if(b.cont=='chest')meta=fdir();else if(b.sign)meta=fdir();
 else if(b.climb){if(t.ny!=0)return;meta=nIdx(t.nx,0,t.nz)}
 else if(b.rs=='lever'||b.rs=='button')meta=nIdx(t.nx,t.ny,t.nz);
 else if(b.rs=='plate'&&t.ny!=1)return;
 else if(b.rs=='piston'){const ax=Math.abs(d[0]),ay=Math.abs(d[1]),az=Math.abs(d[2]);meta=ay>ax&&ay>az?(d[1]>0?2:3):ax>az?(d[0]>0?0:1):(d[2]>0?4:5)}
 if(gm!=2&&(b.s||(b.box&&!b.nocol))){const w=P.w/2;if(x+1>P.x-w&&x<P.x+w&&y+1>P.y&&y<P.y+P.h&&z+1>P.z-w&&z<P.z+w)return}
 edit(x,y,z,s.id,meta);settle(x,y,z);if(s.id==B.sponge){if(absorbWater(x,y,z)){edit(x,y,z,B.wet_sponge,0)}}consume()}
function breakBlock(x,y,z,drop){
 const id=getBlock(x,y,z),b=BL[id];if(!id||b.hard>=1e8)return;const mt=getMeta(x,y,z);
 burst(x+.5,y+.5,z+.5,b.t[2],10,b.tint?(b.tc||[.45,.75,.3]):undefined);
 if(b.cont){const k=kk(x,y,z),c=conts.get(k);if(c){for(const st of c.s)if(st)spawnItem(x+.5,y+.5,z+.5,st.id,st.n);conts.delete(k)}}
 edit(x,y,z,0);signTexts.delete(kk(x,y,z));
 if(b.door){const oy=(mt&8)?y-1:y+1,ob=getBlock(x,oy,z);if(ob&&BL[ob].door)edit(x,oy,z,0)}
 if(drop){if(b.dropm){for(const[di,dn]of b.dropm(mt))if(di)spawnItem(x+.5,y+.5,z+.5,di,dn)}
  else{const dd=b.dropf?b.dropf():(b.drop!==undefined?b.drop:id);let n=1;if(b.dropn)n=b.dropn[0]+Math.floor(Math.random()*(b.dropn[1]-b.dropn[0]+1));if(dd)spawnItem(x+.5,y+.5,z+.5,dd,n)}
  if(b.oreXp)addXp(1+(Math.random()*3|0))}
 settleUp(x,y,z);const up=getBlock(x,y+1,z);if(up&&BL[up].cross)breakBlock(x,y+1,z,true)}
/* ---------- spawn por dimensão ---------- */
function spawnTick(dt){spawnT-=dt;if(spawnT>0)return;spawnT=1;if(!RULES.doMobSpawning)return;
 const hostile=mobs.filter(m=>MT[m.t].host).length,pass=mobs.length-hostile,a=Math.random()*6.283,r=16+Math.random()*16,x=P.x+Math.cos(a)*r,z=P.z+Math.sin(a)*r,c=chunkAt(x,z);if(!c)return;
 const X=Math.floor(x),Z=Math.floor(z);
 if(dim==1){if(gm==0&&!peace&&hostile<14){const yy=Math.floor(P.y-10+Math.random()*20);if(yy>20&&!getBlock(X,yy,Z)&&!getBlock(X,yy+1,Z)&&getBlock(X,yy-1,Z)&&BL[getBlock(X,yy-1,Z)].s)spawnMob(['emberling','wisp','wisp','zombie'][Math.random()*4|0],x,yy,z)}return}
 if(dim==2){if(gm==0&&!peace&&hostile<10){const y=endTop(x,z);if(getBlock(X,y,Z)==B.end_stone&&!getBlock(X,y+1,Z))spawnMob('stalker',x,y+1,z)}return}
 const bio=c.bio[(X&15)+(Z&15)*16];
 if(gm==0&&!peace&&hostile<14){
  if(dayF<.35){const y=groundY(x,z);if(y>0)spawnMob(['zombie','zombie','skeleton','bomber','spider','stalker'][Math.random()*6|0],x,y,z)}
  else{const y=Math.floor(P.y-8+Math.random()*14);if(!getBlock(X,y,Z)&&!getBlock(X,y+1,Z)&&getBlock(X,y-1,Z)&&BL[getBlock(X,y-1,Z)].s&&y<c.hm[(X&15)+(Z&15)*16]-3)spawnMob(['zombie','skeleton','bomber','spider','slime','bat'][Math.random()*6|0],x,y,z)}}
 if(pass<16&&dayF>.4){const y=groundY(x,z);if(y>0&&getBlock(X,y-1,Z)==1){let t=['pig','cow','sheep','chicken','rabbit','horse'][Math.random()*6|0];if([7,9,10,11,15,16,17,18,32].includes(bio)&&Math.random()<.2)t='wolf';if([0,1,4,22,23].includes(bio)&&Math.random()<.06)t='villager';spawnMob(t,x,y,z)}}
 if(pass<16&&bio>=41&&bio<=50&&getBlock(X,SEA-2,Z)==10)spawnMob('fish',x,SEA-2,z)}
/* ---------- mobs extras (criaturas genéricas/originais) ---------- */
Object.assign(MT,{
 rabbit:{hp:3,w:.4,h:.5,sp:2.4,kind:'p',parts:[...quad([.65,.5,.35],[.65,.5,.35],[.55,.42,.3],.4,.25,.25,.15,'face',.22),{s:[.05,.2,.05],o:[-.06,.5,.26],c:[.65,.5,.35]},{s:[.05,.2,.05],o:[.06,.5,.26],c:[.65,.5,.35]}],drops:[[I.chicken,0,1],[I.leather,0,1]]},
 horse:{hp:20,w:1,h:1.6,sp:2.2,kind:'p',parts:[...quad([.55,.38,.22],[.5,.33,.2],[.3,.2,.12],1.5,.65,.7,.8,'face',.5),{s:[.1,.5,.25],o:[0,1.7,.55],c:[.2,.13,.08]},{s:[.1,.1,.6],o:[0,1.05,-.95],c:[.2,.13,.08],tail:1}],drops:[[I.leather,0,2]]},
 bat:{hp:6,w:.4,h:.5,sp:2.5,kind:'p',fly:1,flapAlways:1,parts:[{s:[.2,.25,.15],o:[0,.3,0],c:[.2,.15,.18]},{s:[.2,.18,.18],o:[0,.5,.05],c:[.25,.2,.22],pv:-.09,hd:1},{s:[.4,.02,.25],o:[-.3,.35,0],c:[.15,.1,.12],wing:1},{s:[.4,.02,.25],o:[.3,.35,0],c:[.15,.1,.12],wing:-1}],drops:[]},
 fish:{hp:3,w:.4,h:.3,sp:1.2,kind:'p',swim:1,parts:[{s:[.2,.2,.5],o:[0,.15,0],c:[.85,.5,.3]},{s:[.03,.22,.18],o:[0,.15,-.32],c:[.8,.4,.25],tail:1}],drops:[[I.cod,1,1]]},
 emberling:{hp:4,w:.8,h:.8,sp:3,kind:'h',range:20,dmg:2,slime:1,col:[1,.45,.1],parts:[],drops:[[I.glowdust,0,2]]},
 wisp:{hp:10,w:.6,h:1.2,sp:2,kind:'h',range:30,fly:1,parts:[{s:[.5,.5,.5],o:[0,.9,0],c:[1,.7,.2],f:TN.eface,pv:-.25,hd:1},{s:[.3,.3,.3],o:[0,.9,0],c:[1,.9,.4]},
  ...[[-1,-1],[1,-1],[-1,1],[1,1]].map((q,i)=>({s:[.08,.5,.08],o:[q[0]*.2,.45,q[1]*.2],c:[1,.5,.1],sw:.3,ph:i%2?1:-1}))],drops:[[I.blaze_rod,0,1]]}});
for(const k in MT)MT[k].host=MT[k].kind=='h';
/* ---------- interface de inventário/contêiner ---------- */
function slotS(st,x,y,s,hl,bare){if(!bare){hud.fillStyle=hl?'rgba(255,255,255,.35)':'rgba(30,30,30,.7)';hud.fillRect(x,y,s,s);hud.strokeStyle=hl?'#fff':'#666';hud.lineWidth=hl?3:1;hud.strokeRect(x+.5,y+.5,s-1,s-1)}else if(hl){hud.fillStyle='rgba(255,255,255,.4)';hud.fillRect(x,y,s,s)}
 if(st){icon(st.id,x+(bare?0:4),y+(bare?0:4),s-(bare?0:8));if(st.n>1&&gm!=1)txt(''+st.n,x+s-3,y+s-4,Math.max(10,Math.floor(s/3)),'#fff','right')}}
function slotClick(a,i){const Hd=UI.held;
 if(!Hd){if(a[i]){UI.held={a,i};UI.dragStart={x:UI.mx,y:UI.my}}return}
 const src=Hd.a,si=Hd.i,A=src[si],Bb=a[i];UI.held=null;UI.dragStart=null;if(!A||(src===a&&si===i))return;
 const cf=UI.cont&&UI.cont.t=='furnace'?UI.cont.s:null;
 if(cf&&a===cf&&i==2)return;
 if(cf&&src===cf&&si==2&&Bb&&Bb.id!=A.id)return;
 if(Bb&&Bb.id==A.id){const m=Math.min(A.n,64-Bb.n);Bb.n+=m;A.n-=m;if(!A.n)src[si]=null}else{a[i]=A;src[si]=Bb}}
function dropHeld(){if(!UI.held)return;const{a,i}=UI.held,st=a[i];if(st){a[i]=null;spawnItem(P.x-Math.sin(P.yaw)*.7,P.y+1.2,P.z-Math.cos(P.yaw)*.7,st.id,st.n)}UI.held=null;UI.dragStart=null}
function closeContainer(){if(UI.cont&&UI.cont.t=='crafting')UI.craft3=UI.cont.s;if(UI.held)dropHeld();UI.cont=null;invOpen=false;hc.requestPointerLock()}
const BKEY=Object.fromEntries(Object.entries(B).map(([k,v])=>[v,k]));
function categoryOf(id){
 const key=id<IB?(BKEY[id]||''):((IT[id]&&IT[id].key)||'');
 const nm=String(nameOf(id)).toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'');
 if(/redstone|piston|lever|button|pressure_plate|wire|repeater|comparator|dispenser|observer|hopper|dropper|target|tnt/.test(key+' '+nm))return 'redstone';
 if(/boat|raft|minecart|rail/.test(key+' '+nm))return 'transport';
 if(/pickaxe|axe|shovel|hoe|shears|fishing_rod|flint|bucket|compass|clock|spyglass/.test(key))return 'tools';
 if(/sword|bow|arrow|helmet|chestplate|leggings|boots|shield|trident/.test(key))return 'combat';
 if(/apple|bread|beef|pork|chicken|mutton|fish|salmon|cod|stew|carrot|potato|berries|melon_slice|cookie|cake|pie|food|wheat/.test(key+' '+nm))return 'food';
 if(/ingot|nugget|diamond|emerald|coal|charcoal|quartz|lapis|redstone_dust|string|leather|stick|flint|clay_ball|brick|bone|gunpowder|dye|seeds|wheat|egg|feather|slime|pearl/.test(key+' '+nm))return 'materials';
 if(/sapling|leaves|log|wood|grass|flower|mushroom|dirt|stone|ore|sand|gravel|ice|snow|coral|kelp|seagrass|cactus|vine|root|moss|clay|netherrack|end_stone/.test(key+' '+nm)|| (id<IB&&BL[id].cross))return 'nature';
 if(/wool|concrete|terracotta|stained_glass|carpet|banner|candle/.test(key+' '+nm))return 'colored';
 if(/chest|furnace|crafting|door|trapdoor|fence|gate|sign|ladder|bed|torch|lantern|anvil|enchant|brewing|barrel|bookshelf|lectern|bell|beacon|flower_pot|frame/.test(key+' '+nm))return 'functional';
 if(id<IB&&BL[id].s&&!BL[id].cross&&!BL[id].liq)return 'building';
 return 'misc';
}
const CREATIVE_CATS=[
 ['building','Blocos de construção'],['nature','Blocos naturais'],['functional','Funcionais'],['redstone','Redstone'],['colored','Coloridos'],['tools','Ferramentas'],['combat','Combate'],['food','Comida'],['materials','Materiais'],['transport','Transporte'],['misc','Diversos']
];
const CATIDS=Object.fromEntries(CREATIVE_CATS.map(([k])=>[k,ALLIDS.filter(id=>categoryOf(id)==k)]));
function recipeNeedsTable(r){
 const id=r[0],key=(IT[id]&&IT[id].key)||BKEY[id]||'',nm=String(nameOf(id)).toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'');
 if(/button|botao|pressure_plate|placa de pressao|lever|alavanca|torch|tocha/.test(key+' '+nm))return false;
 return /_(pickaxe|axe|shovel|hoe|sword|helmet|chestplate|leggings|boots|door|trapdoor|fence|fence_gate|stairs|slab|sign|hanging_sign|boat|raft|button|pressure_plate|wall|chest_boat)$/.test(key)||/picareta|machado|pá de|enxada|espada|capacete|peitoral|calças|botas|porta|alçapão|cerca|escada|laje|placa|barco/.test(nm)||r[2].reduce((n,q)=>n+q[1],0)>4;
}
const SHAPELESS_RECIPE_OUTPUTS=new Set([
 B.button, I.clay_ball, I.flint_steel, I.ender_eye,
 ...Object.keys(WOODS).map(w=>B[w+'_planks'])
]);
// Compressão/descompressão de materiais: não dependem de posição na grade.
for(const ik of ['iron_ingot','gold_ingot','diamond','emerald','lapis','redstone','coal','copper_ingot'])if(I[ik])SHAPELESS_RECIPE_OUTPUTS.add(I[ik]);
const RECIPE_SHAPES=new Map([
 [B.stonebr,[[0,0],[0,0]]],[21,[[0,0],[0,0]]],[B.red_sandstone,[[0,0],[0,0]]],[20,[[0,0],[0,0]]],
 [B.quartz_block,[[0,0],[0,0]]],[25,[[0,0],[0,0]]],[23,[[0,1,0],[1,0,1],[0,1,0]]],
 [I.bread,[[0,0,0]]],[B.hay,[[0,0,0],[0,0,0],[0,0,0]]],
 [I.golden_apple,[[1,1,1],[1,0,1],[1,1,1]]],[B.wool_white,[[0,0],[0,0]]],
 [B.ladder,[[0,null,0],[0,0,0],[0,null,0]]],[B.bamboo_mosaic,[[0],[0]]],
 [I.shears,[[0,null],[null,0]]],[B.wall_cobble,[[0,0,0],[0,0,0]]],
 [B.lamp,[[null,0,null],[0,1,0],[null,0,null]]]
]);
function recipePattern(r){
 const id=r[0],key=(IT[id]&&IT[id].key)||BKEY[id]||'',nm=String(nameOf(id)).toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'');
 if(RECIPE_SHAPES.has(id))return RECIPE_SHAPES.get(id);
 const q=r[2],find=(fn)=>q.findIndex(fn),stick=find(a=>a[0]===I.stick),mat=find((a,i)=>i!==stick),one=0;
 const P=(a)=>a;
 if(/planks/.test(key)||/tábuas/.test(nm))return [[0]];
 if(key==='stick'||/graveto/.test(nm))return [[0],[0]];
 if(key==='crafting_table'||/bancada/.test(nm))return [[0,0],[0,0]];
 if(/pickaxe|picareta/.test(key+' '+nm))return [[mat,mat,mat],[null,stick,null],[null,stick,null]];
 if(/sword|espada/.test(key+' '+nm))return [[mat],[mat],[stick]];
 if(/shovel|pá de/.test(key+' '+nm))return [[mat],[stick],[stick]];
 if(/axe|machado/.test(key+' '+nm))return [[mat,mat],[mat,stick],[null,stick]];
 if(/hoe|enxada/.test(key+' '+nm))return [[mat,mat],[null,stick],[null,stick]];
 if(/helmet|capacete/.test(key+' '+nm))return [[mat,mat,mat],[mat,null,mat]];
 if(/chestplate|peitoral/.test(key+' '+nm))return [[mat,null,mat],[mat,mat,mat],[mat,mat,mat]];
 if(/leggings|calças/.test(key+' '+nm))return [[mat,mat,mat],[mat,null,mat],[mat,null,mat]];
 if(/boots|botas/.test(key+' '+nm))return [[mat,null,mat],[mat,null,mat]];
 if(/door|porta/.test(key+' '+nm))return [[0,0],[0,0],[0,0]];
 if(/trap_iron/.test(key))return [[0,0],[0,0]];if(/trap|alçapão/.test(key+' '+nm))return [[0,0,0],[0,0,0]];
 if(/stair|escada/.test(key+' '+nm))return [[0,null,null],[0,0,null],[0,0,0]];
 if(/slab|laje/.test(key+' '+nm))return [[0,0,0]];
 if(/fence_gate|portão/.test(key+' '+nm))return [[stick,mat,stick],[stick,mat,stick]];
 if(/fence|cerca/.test(key+' '+nm))return [[mat,stick,mat],[mat,stick,mat]];
 if(/hanging_sign/.test(key))return [[1,null,1],[0,0,0],[0,0,0]];
 if(/sign|placa/.test(key+' '+nm))return [[0,0,0],[0,0,0],[null,1,null]];
 if(/boat|raft|barco|jangada/.test(key+' '+nm))return [[null,null,null],[0,null,0],[0,0,0]];
 if(key==='chest'||/baú/.test(nm))return [[0,0,0],[0,null,0],[0,0,0]];
 if(key==='furnace'||/fornalha/.test(nm))return [[0,0,0],[0,null,0],[0,0,0]];
 if(key==='bookshelf'||/estante/.test(nm))return [[0,0,0],[1,1,1],[0,0,0]];
 if(/lã|la /.test(nm)||key==='wool_white')return [[0,0],[0,0]];
 if(/hay|fardo/.test(key+' '+nm))return [[0,0,0],[0,0,0],[0,0,0]];
 if(/golden_apple|maçã dourada/.test(key+' '+nm))return [[1,1,1],[1,0,1],[1,1,1]];
 if(/glass_pane|iron_bars|painel de vidro|barras de ferro/.test(key+' '+nm))return [[0,0,0],[0,0,0]];
 if(/ladder|escada de mão/.test(key+' '+nm))return [[0,null,0],[0,0,0],[0,null,0]];
 if(/bucket|balde/.test(key+' '+nm))return [[0,null,0],[null,0,null]];
 if(/bed|cama/.test(key+' '+nm))return [[0,0,0],[1,1,1]];
 if(/bread|pão/.test(key+' '+nm))return [[0,0,0]];
 if(/wall|muro/.test(key+' '+nm))return [[0,0,0],[0,0,0]];
 if(/torch|tocha/.test(key+' '+nm))return [[1],[0]];
 if(/lever|alavanca/.test(key+' '+nm))return [[0],[1]];
 if(/button|botão/.test(key+' '+nm))return [[0]];
 if(/plate|placa de pressão/.test(key+' '+nm))return [[0,0]];
 if(/redstone_torch/.test(key))return [[1],[0]];
 if(/piston|pistão/.test(key+' '+nm))return [[0,0,0],[1,2,1],[1,3,1]];
 if(/hay|fardo/.test(key+' '+nm))return [[0,0,0],[0,0,0],[0,0,0]];
 if(/block/.test(key)&&q.length===1&&q[0][1]===9)return [[0,0,0],[0,0,0],[0,0,0]];
 if(q.length===1&&q[0][1]===1)return [[0]];
 return null;
}
function findPatternCells(grid,r){
 if(SHAPELESS_RECIPE_OUTPUTS.has(r[0]))return null;const pat=recipePattern(r);if(!pat)return false;const n=Math.sqrt(grid.length)|0,ph=pat.length,pw=Math.max(...pat.map(row=>row.length));
 const q=r[2],matches=(st,qi)=>{if(!st)return false;const v=q[qi][0];return typeof v==='string'?(TAG[v]||[]).includes(st.id):st.id===v};
 for(let mirror=0;mirror<2;mirror++)for(let oy=0;oy<=n-ph;oy++)for(let ox=0;ox<=n-pw;ox++){
  let ok=true,cells=[];for(let y=0;y<n&&ok;y++)for(let x=0;x<n;x++){
   let py=y-oy,px=x-ox,token=null;
   if(py>=0&&py<ph){const row=pat[py],ix=mirror?(row.length-1-px):px;if(ix>=0&&ix<row.length)token=row[ix]}
   const idx=y*n+x,st=grid[idx];if(token===null||token===undefined){if(st){ok=false;break}}else if(!matches(st,token)){ok=false;break}else cells.push(idx);
  }
  if(ok)return cells;
 }
 return false;
}
function patternFits(grid,r){return findPatternCells(grid,r)!==false;}
function recipeForGrid(grid,table){
 const counts=new Map();let total=0;
 for(const st of grid){if(st&&st.n>0){counts.set(st.id,(counts.get(st.id)||0)+st.n);total+=st.n}}
 if(!total)return null;
 for(const r of RECIPES){if(recipeNeedsTable(r)&&!table)continue;if(!table){const pat=recipePattern(r);if(pat&&(pat.length>2||Math.max(...pat.map(row=>row.length))>2))continue}const left=new Map(counts);let ok=true,need=0;
  for(const q of r[2]){let n=q[1];need+=n;const ids=typeof q[0]=='string'?(TAG[q[0]]||[]):[q[0]];for(const id of ids){const have=left.get(id)||0,take=Math.min(have,n);if(take){left.set(id,have-take);n-=take}if(!n)break}if(n){ok=false;break}}
  const allowed=new Set(r[2].flatMap(q=>typeof q[0]=='string'?(TAG[q[0]]||[]):[q[0]]));if(ok&&[...left].every(([id,n])=>n===0||allowed.has(id))&&patternFits(grid,r))return r;
 }
 return null;
}
function craftGrid(grid,table){
 const r=recipeForGrid(grid,table);if(!r)return;const cells=findPatternCells(grid,r);
 if(cells!==null&&cells!==false){for(const i of cells){const st=grid[i];if(st){st.n--;if(!st.n)grid[i]=null}}}
 else{for(const q of r[2]){let n=q[1],ids=typeof q[0]=='string'?(TAG[q[0]]||[]):[q[0]];for(const id of ids){for(let i=grid.length-1;i>=0&&n>0;i--){const st=grid[i];if(st&&st.id==id){const take=Math.min(st.n,n);st.n-=take;n-=take;if(!st.n)grid[i]=null}}if(!n)break}}}
 const left=addInv(r[0],r[1]);if(left)spawnItem(P.x,P.y+1,P.z,r[0],left);
}
function drawCraftGrid(grid,cols,x,y,sz,table){
 for(let i=0;i<grid.length;i++){const xx=x+(i%cols)*sz,yy=y+Math.floor(i/cols)*sz;slotS(grid[i],xx,yy,sz,!!(UI.held&&UI.held.a===grid&&UI.held.i==i),false);UI.rects.push({x:xx,y:yy,w:sz,h:sz,fn:()=>slotClick(grid,i)})}
 const r=recipeForGrid(grid,table),rx=x+cols*sz+sz*.65,ry=y+Math.floor(cols==2?0:1)*sz*.5;
 txt('→',rx-sz*.35,ry+sz*.65,sz*.7,'#ddd','center');slotS(r?{id:r[0],n:r[1]}:null,rx,ry,sz*1.15,false,false);
 if(r)UI.rects.push({x:rx,y:ry,w:sz*1.15,h:sz*1.15,fn:()=>craftGrid(grid,table)});
}
function drawJavaInventory(){
 const g=hud,cont=UI.cont,im=cont&&cont.t==='crafting'?JAVA_GUI.craft:JAVA_GUI.inv;if(!im)return false;
 const scale=Math.max(.85,Math.min(2.35,W/190,HH/178)),sw=176*scale,sh=166*scale,x0=Math.floor((W-sw)/2),y0=Math.floor((HH-sh)/2);
 g.imageSmoothingEnabled=false;g.drawImage(im,0,0,176,166,x0,y0,sw,sh);
 const label=(t,x,y)=>{g.font=`${8*scale}px monospace`;g.textAlign='left';g.fillStyle='#404040';g.fillText(t,x0+x*scale,y0+y*scale)};
 const drawStack=(arr,i,x,y,click=true)=>{const st=arr[i];if(st){icon(st.id,x+scale,y+scale,16*scale);if(st.n>1){g.font=`${8*scale}px monospace`;g.textAlign='right';g.lineWidth=Math.max(2,2*scale);g.strokeStyle='#3f3f3f';g.strokeText(String(st.n),x+17*scale,y+16*scale);g.fillStyle='#fff';g.fillText(String(st.n),x+17*scale,y+16*scale)}}if(click)UI.rects.push({x,y,w:18*scale,h:18*scale,fn:()=>slotClick(arr,i)});if(st&&UI.mx>=x&&UI.mx<=x+18*scale&&UI.my>=y&&UI.my<=y+18*scale)UI.hover=st.id};
 const drawEmpty=(x,y,fn)=>UI.rects.push({x,y,w:18*scale,h:18*scale,fn});
 if(cont&&cont.t==='crafting'){
  label('Fabricação',30,13);label('Inventário',8,75);
  for(let r=0;r<3;r++)for(let c=0;c<3;c++)drawStack(cont.s,r*3+c,x0+(30+18*c)*scale,y0+(17+18*r)*scale);
  const result=recipeForGrid(cont.s,true),rx=x0+124*scale,ry=y0+35*scale;
  if(result)icon(result[0],rx+scale,ry+scale,16*scale);if(result&&result[1]>1){g.font=`${8*scale}px monospace`;g.textAlign='right';g.fillStyle='#fff';g.fillText(String(result[1]),rx+17*scale,ry+16*scale)}
  drawEmpty(rx,ry,()=>{if(result)craftGrid(cont.s,true)});
  for(let r=0;r<3;r++)for(let c=0;c<9;c++)drawStack(inv,9+r*9+c,x0+(8+18*c)*scale,y0+(84+18*r)*scale);
  for(let c=0;c<9;c++)drawStack(inv,c,x0+(8+18*c)*scale,y0+142*scale);
  return true;
 }
 label('Fabricação',88,16);label('Inventário',8,75);
 // Armadura e mão secundária seguem as posições originais do inventário Java.
 for(let i=0;i<4;i++){const x=x0+8*scale,y=y0+(8+18*i)*scale,st=armor[i]?{id:armor[i],n:1}:null;if(st)icon(st.id,x+scale,y+scale,16*scale);UI.rects.push({x,y,w:18*scale,h:18*scale,fn:()=>{const hd=UI.held&&UI.held.a[UI.held.i];if(hd&&IT[hd.id]?.armor===i){const old=armor[i];armor[i]=hd.id;hd.n--;if(!hd.n)UI.held.a[UI.held.i]=null;UI.held=null;if(old)addInv(old,1)}}})}
 const ox=x0+77*scale,oy=y0+62*scale;if(UI.offhand[0])icon(UI.offhand[0].id,ox+scale,oy+scale,16*scale);UI.rects.push({x:ox,y:oy,w:18*scale,h:18*scale,fn:()=>slotClick(UI.offhand,0)});
 for(let r=0;r<2;r++)for(let c=0;c<2;c++)drawStack(UI.craft2,r*2+c,x0+(88+18*c)*scale,y0+(26+18*r)*scale);
 const result=recipeForGrid(UI.craft2,false),rx=x0+144*scale,ry=y0+35*scale;if(result)icon(result[0],rx+scale,ry+scale,16*scale);if(result&&result[1]>1){g.font=`${8*scale}px monospace`;g.textAlign='right';g.fillStyle='#fff';g.fillText(String(result[1]),rx+17*scale,ry+16*scale)}
 drawEmpty(rx,ry,()=>{if(result)craftGrid(UI.craft2,false)});
 for(let r=0;r<3;r++)for(let c=0;c<9;c++)drawStack(inv,9+r*9+c,x0+(8+18*c)*scale,y0+(84+18*r)*scale);
 for(let c=0;c<9;c++)drawStack(inv,c,x0+(8+18*c)*scale,y0+142*scale);
 return true;
}
function drawInv(){
 const g=hud,sz=Math.max(30,Math.min(42,Math.floor(W/24))),cont=UI.cont;g.fillStyle='rgba(0,0,0,.52)';g.fillRect(0,0,W,HH);
 if(((!cont&&gm==0)||(cont&&cont.t==='crafting'))&&drawJavaInventory()){
  if(UI.held){const st=UI.held.a[UI.held.i];if(st)icon(st.id,UI.mx-16,UI.my-16,32)}
  const h=UI.hover;if(h!==undefined&&h>=0)txt(nameOf(h),UI.mx+14,UI.my-6,15,'#fff');
  return;
 }

 let hover=-1;const over=(x,y,w,h)=>UI.mx>x&&UI.mx<x+w&&UI.my>y&&UI.my<y+h;
 const S=(arr,i,x,y,s)=>{const st=arr[i],hl=UI.held&&UI.held.a===arr&&UI.held.i==i;slotS(st,x,y,s,hl,false);UI.rects.push({x,y,w:s,h:s,fn:()=>slotClick(arr,i)});if(st&&over(x,y,s,s))hover=st.id};
 const drawPlayerSlots=(x,y,includeHot=true)=>{txt('Inventário',x,y-12,15);for(let k=0;k<27;k++)S(inv,k+9,x+(k%9)*sz,y+Math.floor(k/9)*sz,sz);if(includeHot){const hy=y+3*sz+10;for(let i=0;i<9;i++)S(inv,i,x+i*sz,hy,sz);return hy+sz}return y+3*sz};
 let armPos=i=>({x:W/2-5.4*sz,y:HH*.48+i*sz});
 if(!cont&&gm==1){
  const left=Math.max(145,Math.min(205,W*.17)),lx=Math.max(10,W/2-6.8*sz),top=HH*.12;
  txt('CRIATIVO',lx,top-20,18,'#fff');
  CREATIVE_CATS.forEach(([key,label],i)=>{const y=top+i*31,active=UI.creativeCat===key;g.fillStyle=active?'rgba(95,125,85,.95)':'rgba(45,45,45,.92)';g.fillRect(lx,y,left,27);g.strokeStyle=active?'#d7efb8':'#777';g.strokeRect(lx+.5,y+.5,left-1,26);txt((active?'▼ ':'▶ ')+label,lx+8,y+18,13,active?'#fff':'#ddd');UI.rects.push({x:lx,y,w:left,h:27,fn:()=>{UI.creativeCat=key;UI.scroll=0}})});
  const items=CATIDS[UI.creativeCat]||[],gx=lx+left+18,gy=top,cols=Math.max(5,Math.min(9,Math.floor((W-gx-20)/sz))),rows=Math.max(4,Math.floor((HH-gy-90)/sz));
  txt(CREATIVE_CATS.find(c=>c[0]===UI.creativeCat)?.[1]||'Itens',gx,gy-12,15);
  for(let k=0;k<cols*rows;k++){const id=items[k+UI.scroll*cols];if(id===undefined)break;const x=gx+(k%cols)*sz,y=gy+Math.floor(k/cols)*sz;slotS({id,n:64},x,y,sz,false,false);if(over(x,y,sz,sz))hover=id;UI.rects.push({x,y,w:sz,h:sz,fn:()=>{inv[sel]={id,n:64}}})}
  txt(`${items.length} itens`,gx,gy+rows*sz+18,12,'#bbb');
 }else if(cont&&cont.t=='crafting'){
  const gx=Math.floor(W/2-4.5*sz),gy=HH*.13;txt('Bancada de fabricação',gx,gy-18,18);
  drawCraftGrid(cont.s,3,gx+sz*1.1,gy+sz*.35,sz,true);
  const invY=gy+4.5*sz;drawPlayerSlots(gx,invY,true);
  for(let i=0;i<4;i++){const p=armPos(i);g.fillStyle='rgba(30,30,60,.8)';g.fillRect(p.x,p.y,sz,sz);slotS(armor[i]?{id:armor[i],n:1}:null,p.x,p.y,sz,false,false);UI.rects.push({x:p.x,y:p.y,w:sz,h:sz,fn:()=>{const hd=UI.held&&UI.held.a[UI.held.i];if(hd&&IT[hd.id]?.armor===i){const old=armor[i];armor[i]=hd.id;hd.n--;if(!hd.n)UI.held.a[UI.held.i]=null;UI.held=null;if(old)addInv(old,1)}}})}
 }else if(cont){
  const gx=Math.floor(W/2-4.5*sz),gy=HH*.10;txt(cont.t=='doubleChest'?'Baú duplo':cont.t=='chest'?'Baú':'Fornalha',gx,gy-12,16);let my;
  if(cont.t=='chest'){for(let k=0;k<27;k++)S(cont.s,k,gx+(k%9)*sz,gy+(k/9|0)*sz,sz);my=gy+3*sz+16}
  else if(cont.t=='doubleChest'){for(let k=0;k<54;k++){const a=k<27?cont.a.s:cont.b.s,i=k%27;S(a,i,gx+(i%9)*sz,gy+((k/9)|0)*sz,sz)}my=gy+6*sz+16}
  else{S(cont.s,0,gx+3*sz,gy,sz);S(cont.s,1,gx+3*sz,gy+2*sz,sz);S(cont.s,2,gx+5.5*sz,gy+sz,sz);g.fillStyle='#444';g.fillRect(gx+3*sz+4,gy+sz+4,sz-8,sz*.25);g.fillStyle='#fa3';g.fillRect(gx+3*sz+4,gy+sz+4,(sz-8)*Math.min(1,cont.prog/10),sz*.25);g.fillStyle='#d62';g.fillRect(gx+3*sz+4,gy+sz*1.6,(sz-8)*Math.max(0,Math.min(1,cont.burn/cont.bm)),sz*.2);my=gy+3.4*sz}
  for(let k=9;k<36;k++)S(inv,k,gx+((k-9)%9)*sz,my+((k-9)/9|0)*sz,sz);for(let i=0;i<9;i++)S(inv,i,gx+i*sz,my+3*sz+10,sz);
 }else if(gm==0){
  const gx=Math.floor(W/2-4.5*sz),gy=HH*.10;txt('Inventário',gx,gy-15,17);
  // Armadura e mão secundária no estilo Java.
  for(let i=0;i<4;i++){const x=gx-sz-18,y=gy+i*(sz+4);slotS(armor[i]?{id:armor[i],n:1}:null,x,y,sz,false,false);UI.rects.push({x,y,w:sz,h:sz,fn:()=>{const hd=UI.held&&UI.held.a[UI.held.i];if(hd&&IT[hd.id]?.armor===i){const old=armor[i];armor[i]=hd.id;hd.n--;if(!hd.n)UI.held.a[UI.held.i]=null;UI.held=null;if(old)addInv(old,1)}}})}
  const offX=gx-sz-18,offY=gy+4*(sz+4);slotS(UI.offhand[0],offX,offY,sz,false,false);UI.rects.push({x:offX,y:offY,w:sz,h:sz,fn:()=>slotClick(UI.offhand,0)});txt('Offhand',offX,offY+sz+14,10,'#aaa');
  const invEnd=drawPlayerSlots(gx,gy+4.2*sz,true);
  const cx=gx+10*sz+12,cy=gy+sz*.35;txt('Fabricação',cx,cy-12,14);drawCraftGrid(UI.craft2,2,cx,cy,sz,false);
  txt('Armadura: '+armorPts(),gx,invEnd+18,12,'#9bf');
 }else{
  txt(gm==2?'Espectador':'Inventário',W/2,HH*.2,18,'#fff','center');
 }
 if(UI.held){const st=UI.held.a[UI.held.i];if(st)icon(st.id,UI.mx-sz/2,UI.my-sz/2,sz)}
 if(!invOpen){const hg=hotG();for(let i=0;i<9;i++){const r=hg.rx(i);UI.rects.push({x:r.x,y:r.y,w:r.w,h:r.h,fn:()=>{if(gm==0||cont)slotClick(inv,i);else sel=i}})}}
 if(hover>=0)txt(nameOf(hover),UI.mx+14,UI.my-6,15,'#ff8');
}
/* ---------- comandos ---------- */
const nzs=s=>s.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'');
function locate(name){const q=nzs(name),bi=BIOMES.findIndex(b=>nzs(b).includes(q));if(bi<0||dim)return null;
 for(let r=16;r<5000;r+=32){const n=Math.max(8,r/16|0);for(let a=0;a<n;a++){const th=a/n*6.2832,x=Math.round(P.x+Math.cos(th)*r),z=Math.round(P.z+Math.sin(th)*r);if(bi>=52&&bi<=55){for(const y of[12,24,32,44]){const caveWet=n2(x/34,z/34,202),caveRock=n2(x/27,z/27,203),deep=n3(x/28,y/18,z/28,204),got=y<22&&deep>.61?54:y<40&&caveRock>.72?55:caveWet>.62?52:caveRock>.5?53:-1;if(got==bi&&cave(x,y,z))return[x,z]}}else if(col(x,z)[3]==bi)return[x,z]}}return null}
function saveWorld(){try{const o={v:3,seed:SEED,tod,dim,xp,gm,pos:[P.x,P.y,P.z],inv,armor,offhand:UI.offhand,craft2:UI.craft2,craft3:UI.craft3,sp:SP,signs:[...signTexts],dims:DIMS.map(D=>({mods:[...D.mods].map(([k,m])=>[k,[...m]]),conts:[...D.conts]}))};const s=JSON.stringify(o);localStorage.setItem('mcjs_save',s);msg('Mundo salvo ('+(s.length/1024|0)+' KB)')}catch(e){msg('Falha ao salvar (cota cheia?)')}}
function loadWorld(){try{const o=JSON.parse(localStorage.getItem('mcjs_save'));if(!o){msg('Nenhum save encontrado');return}
 if(o.seed!==SEED)msg('Aviso: seed do save = '+o.seed+'. Defina window.MC_SEED='+o.seed+' e cole de novo para o mesmo terreno.');
 DIMS.forEach((D,i)=>{for(const c of D.chunks.values()){freeMesh(c.mo);freeMesh(c.mt)}D.chunks.clear();D.rss.clear();D.flq.clear();D.tks.clear();D.mods=new Map(o.dims[i].mods.map(([k,m])=>[k,new Map(m)]));D.conts=new Map(o.dims[i].conts)});
 signTexts.clear();for(const[k,v]of(o.signs||[]))signTexts.set(k,v);dim=o.dim;chunks=DIMS[dim].chunks;mods=DIMS[dim].mods;conts=DIMS[dim].conts;RSS=DIMS[dim].rss;FLQ=DIMS[dim].flq;TKS=DIMS[dim].tks;
 tod=o.tod;xp=o.xp;SP=o.sp;inv.fill(null);o.inv.forEach((s,i)=>inv[i]=s);o.armor.forEach((a,i)=>armor[i]=a);UI.offhand=o.offhand||[null];if(o.craft2)UI.craft2=o.craft2;if(o.craft3)UI.craft3=o.craft3;[P.x,P.y,P.z]=o.pos;mobs.length=0;msg('Mundo carregado')}catch(e){msg('Falha ao carregar')}}
const HELP=['/gamemode 0|1|2 · /time set|add dia|noite|n · /weather clear|rain|thunder · /tp x y z (aceita ~) · /give item [n] · /summon mob [n]','/kill · /heal · /feed · /clear · /difficulty peaceful|easy|normal|hard · /gamerule regra 0|1 · /effect nome seg [amp]','/xp add n · /setblock x y z bloco · /fill x1 y1 z1 x2 y2 z2 bloco · /clone x1 y1 z1 x2 y2 z2 dx dy dz','/dimension overworld|nether|end · /locate bioma · /biome · /spawnpoint · /fly · /speed n · /fov n · /sens n · /rd n','/save · /load · /skin · /pack [bedrock|arquivos] · /view · /armor · /say txt · /me txt · /list · /seed · /summon (lista mobs)'];
function cmd(s){s=(s||'').trim();if(!s)return;if(s[0]!='/'){msg('<Você> '+s);return}
 const a=s.slice(1).split(/\s+/),c=a[0].toLowerCase(),num=(v,b)=>v&&v[0]=='~'?b+(+v.slice(1)||0):+v;
 const blockId=q=>{const id=findItem(q||'');return id!=null&&id<IB?id:null};
 switch(c){
 case'gamemode':case'gm':{const m={0:0,s:0,survival:0,sobrevivencia:0,1:1,c:1,creative:1,criativo:1,2:2,sp:2,spectator:2,espectador:2}[(a[1]||'').toLowerCase()];if(m===undefined)msg('Uso: /gamemode 0|1|2');else setGM(m);break}
 case'time':{const v=(a[2]||a[1]||'').toLowerCase(),T={day:.3,dia:.3,noon:.25,meio:.25,night:.75,noite:.75,midnight:.75};
  if(a[1]=='add')tod=(tod+(+a[2]||0)/24000)%1;else if(a[1]=='query')msg('Hora: '+Math.floor(tod*24000));else tod=T[v]!==undefined?T[v]:isNaN(+v)?tod:(+v>1?+v/24000:+v)%1;msg('Hora ajustada');break}
 case'weather':{const v=(a[1]||'').toLowerCase();weather=v=='rain'||v=='chuva'?1:v=='thunder'||v=='tempestade'?2:0;wT=300;break}
 case'tp':case'teleport':{const x=num(a[1],P.x),y=num(a[2],P.y),z=num(a[3],P.z);if([x,y,z].some(isNaN))msg('Uso: /tp x y z');else{P.x=x;P.y=y;P.z=z;P.vy=0}break}
 case'give':{const id=findItem(a[1]||'');if(id==null)msg('Item não encontrado');else{addInv(id,+a[2]||64);msg('Recebeu '+nameOf(id))}break}
 case'summon':{const t=(a[1]||'').toLowerCase();if(MT[t]){for(let i=0;i<Math.min(20,+a[2]||1);i++)spawnMob(t,P.x-Math.sin(P.yaw)*3+(Math.random()-.5)*2,P.y+1,P.z-Math.cos(P.yaw)*3+(Math.random()-.5)*2)}else msg('Mobs: '+Object.keys(MT).join(', '));break}
 case'kill':mobs.length=0;msg('Mobs removidos');break;
 case'heal':P.hp=20;break;case'feed':P.food=20;break;
 case'clear':inv.fill(null);armor.fill(0);break;
 case'peaceful':case'difficulty':{const v=(a[1]||'').toLowerCase();diff=v=='peaceful'||c=='peaceful'?0:v=='easy'?1:v=='hard'?3:2;peace=diff==0;if(peace)mobs.splice(0,mobs.length,...mobs.filter(m=>!MT[m.t].host));msg('Dificuldade: '+['pacífico','fácil','normal','difícil'][diff]);break}
 case'gamerule':{const r=Object.keys(RULES).find(k=>k.toLowerCase()==(a[1]||'').toLowerCase());if(!r)msg('Regras: '+Object.keys(RULES).join(', '));else{RULES[r]=(a[2]=='false'||a[2]=='0')?0:1;msg(r+' = '+RULES[r])}break}
 case'effect':{const n=(a[1]||'').toLowerCase();const N={speed:'speed',velocidade:'speed',jump:'jump',pulo:'jump',regen:'regen',regeneration:'regen',resistance:'resist',resist:'resist',strength:'strength',forca:'strength',haste:'haste',night_vision:'nv',nv:'nv',slow_falling:'slowfall',slowfall:'slowfall',fire_resistance:'fireres',fireres:'fireres',water_breathing:'waterb',waterb:'waterb',levitation:'lev',lev:'lev',poison:'poison',hunger:'hunger'}[n];
  if(n=='clear'){P.fx={};msg('Efeitos removidos')}else if(!N)msg('Efeitos: speed jump regen resist strength haste nv slowfall fireres waterb lev poison hunger clear');else{P.fx[N]={t:+a[2]||30,a:+a[3]||1};msg('Efeito '+N)}break}
 case'xp':case'experience':{if(a[1]=='add'||!isNaN(+a[1])){xp+=+(a[2]||a[1]);msg('XP: '+xp)}else if(a[1]=='set'){xp=+a[2]||0}break}
 case'setblock':{const x=Math.floor(num(a[1],P.x)),y=Math.floor(num(a[2],P.y)),z=Math.floor(num(a[3],P.z)),id=a[4]=='air'?0:blockId(a[4]);if(id==null||isNaN(x+y+z))msg('Uso: /setblock x y z bloco [meta]');else edit(x,y,z,id,+a[5]||0);break}
 case'fill':{const v=a.slice(1,7).map((s,i)=>Math.floor(num(s,[P.x,P.y,P.z][i%3]))),id=a[7]=='air'?0:blockId(a[7]);if(v.some(isNaN)||id==null){msg('Uso: /fill x1 y1 z1 x2 y2 z2 bloco');break}
  const[x1,y1,z1,x2,y2,z2]=v;let n=0;for(let x=Math.min(x1,x2);x<=Math.max(x1,x2);x++)for(let y=Math.min(y1,y2);y<=Math.max(y1,y2);y++)for(let z=Math.min(z1,z2);z<=Math.max(z1,z2);z++){if(++n>32768){msg('Área grande demais (máx 32768)');return}setBlock(x,y,z,id)}msg(n+' blocos');break}
 case'clone':{const v=a.slice(1,10).map((s,i)=>Math.floor(num(s,[P.x,P.y,P.z][i%3])));if(v.length<9||v.some(isNaN)){msg('Uso: /clone x1 y1 z1 x2 y2 z2 dx dy dz');break}
  const[x1,y1,z1,x2,y2,z2,dx,dy,dz]=v,xs=Math.min(x1,x2),ys=Math.min(y1,y2),zs=Math.min(z1,z2),w=Math.abs(x2-x1)+1,h=Math.abs(y2-y1)+1,l=Math.abs(z2-z1)+1;if(w*h*l>32768){msg('Área grande demais');break}
  const buf=[];for(let x=0;x<w;x++)for(let y=0;y<h;y++)for(let z=0;z<l;z++)buf.push([x,y,z,getBlock(xs+x,ys+y,zs+z),getMeta(xs+x,ys+y,zs+z)]);for(const b of buf)setBlock(dx+b[0],dy+b[1],dz+b[2],b[3],b[4]);msg(buf.length+' blocos clonados');break}
 case'fly':if(gm==2)break;P.fly=!P.fly;msg('Voo: '+P.fly);break;
 case'speed':P.speedMul=Math.max(.2,Math.min(8,+a[1]||1));msg('Velocidade x'+P.speedMul);break;
 case'fov':FOVB=Math.max(40,Math.min(110,+a[1]||70));break;
 case'sens':SENS=Math.max(.0005,Math.min(.01,(+a[1]||2.2)/1000));break;
 case'spawnpoint':case'setworldspawn':if(!dim){SP=[P.x,P.y,P.z];msg('Spawn definido')}break;
 case'dimension':case'dim':{const v=(a[1]||'').toLowerCase(),n=v=='nether'||v=='1'?1:v=='end'||v=='2'?2:v=='overworld'||v=='0'?0:-1;if(n<0)msg('Uso: /dimension overworld|nether|end');else if(n!=dim)goDim(n,n==2?.5:(dim==1?P.x*8:P.x),n==2?.5:(dim==1?P.z*8:P.z),false);break}
 case'locate':{const r=locate(a.slice(1).join(' ')||'');msg(r?'Encontrado em x='+r[0]+' z='+r[1]:'Bioma não encontrado (só no Overworld). Biomas: '+BIOMES.slice(0,56).join(', '));break}
 case'biome':{const cc=chunkAt(P.x,P.z),xx=Math.floor(P.x)&15,zz=Math.floor(P.z)&15,yy=Math.max(0,Math.min(H-1,Math.floor(P.y)));msg('Bioma: '+(cc?BIOMES[cc.cbio[xx+16*(zz+16*yy)]]:'?'));break}
 case'enchant':msg('Encantamentos ainda não existem nesta versão');break;
 case'say':case'tellraw':case'title':msg('[Servidor] '+a.slice(1).join(' '));break;
 case'me':msg('* Você '+a.slice(1).join(' '));break;
 case'list':msg('Jogadores: 1/1 (você)');break;
 case'rd':RD=Math.max(2,Math.min(10,+a[1]||RD));msg('Distância: '+RD);break;
 case'save':saveWorld();break;case'load':loadWorld();break;
 case'skin':importSkin();break;case'pack':case'texturas':if(a[1]=='bedrock'||a[1]=='vanilla')loadVanillaBedrock();else importPack(a[1]=='arquivos');break;
 case'view':view=(view+1)%3;break;
 case'armor':AK.forEach(([k],i)=>armor[i]=I['diamond_'+k]);msg('Armadura de diamante equipada');break;
 case'seed':msg('Seed: '+SEED);break;
 case'help':case'?':for(const l of HELP)msg(l);break;
 default:msg('Comando desconhecido. /help')}}
 
/* ================= CARREGADORES (seus arquivos locais, sem rede) ================= */
const GUI={};let skinUser=false;
function pickFiles(accept,multi,dir,cb){const i=document.createElement('input');i.type='file';i.accept=accept;if(multi)i.multiple=true;if(dir)i.webkitdirectory=true;i.style.display='none';document.body.append(i);
 i.onchange=()=>{const f=[...i.files];i.remove();cb(f)};i.click()}
const loadImg=f=>new Promise((res,rej)=>{const im=new Image(),u=URL.createObjectURL(f);im.onload=()=>{URL.revokeObjectURL(u);res(im)};im.onerror=rej;im.src=u});
function paintTile(t,bm,over,fr,saveSource=true){const i=TN[t],ox=(i%TPR)*16,oy=(i/TPR|0)*16;ac.imageSmoothingEnabled=false;ac.clearRect(ox,oy,16,16);const frameH=bm.height>=bm.width?bm.width:bm.height;ac.drawImage(bm,0,(fr||0)*frameH,bm.width,frameH,ox,oy,16,16);
 if(t=='water'){const d=ac.getImageData(ox,oy,16,16);for(let k=0;k<d.data.length;k+=4){const a=d.data[k+3];if(a<8)continue;const lum=(d.data[k]*.3+d.data[k+1]*.59+d.data[k+2]*.11)/255;d.data[k]=Math.min(255,12+lum*35);d.data[k+1]=Math.min(255,55+lum*95);d.data[k+2]=Math.min(255,125+lum*125);d.data[k+3]=Math.min(185,a*.72)}ac.putImageData(d,ox,oy)}
 if(saveSource&&(t=='water'||t=='lava'))FLUID_SRC[t]=ac.getImageData(ox,oy,16,16);
 if(t=='gs'&&over){const tmp=document.createElement('canvas');tmp.width=tmp.height=16;const g=tmp.getContext('2d');g.imageSmoothingEnabled=false;g.drawImage(over,0,0,over.width,over.width,0,0,16,16);
  const d=g.getImageData(0,0,16,16);for(let k=0;k<d.data.length;k+=4){d.data[k]*=.5;d.data[k+1]*=.8;d.data[k+2]*=.3}g.putImageData(d,0,0);ac.drawImage(tmp,ox,oy)}}
const GUIRX={hf:/hud\/heart\/full\.png$/,hh:/hud\/heart\/half\.png$/,hc:/hud\/heart\/container\.png$/,ff:/hud\/food_full\.png$/,fh:/hud\/food_half\.png$/,fe:/hud\/food_empty\.png$/,af:/hud\/armor_full\.png$/,ah:/hud\/armor_half\.png$/,ae:/hud\/armor_empty\.png$/,
 hb:/hud\/hotbar\.png$/,hs:/hotbar_selection\.png$/,ic:/gui\/icons\.png$/,wg:/gui\/widgets\.png$/,inv:/container\/inventory\.png$/,craft:/container\/crafting_table\.png$/,chest:/container\/generic_54\.png$/};
const JAVA_GUI={},JAVA_GUI_BASE='https://raw.githubusercontent.com/InventivetalentDev/minecraft-assets/26.1/assets/minecraft/textures/gui/';
function loadJavaGui(){
 const files={
  inv:'container/inventory.png',craft:'container/crafting_table.png',chest:'container/generic_54.png',
  hb:'sprites/hud/hotbar.png',hs:'sprites/hud/hotbar_selection.png',cross:'sprites/hud/crosshair.png',
  heartFull:'sprites/hud/heart/full.png',heartHalf:'sprites/hud/heart/half.png',heartEmpty:'sprites/hud/heart/container.png',
  foodFull:'sprites/hud/food_full.png',foodHalf:'sprites/hud/food_half.png',foodEmpty:'sprites/hud/food_empty.png',
  armorFull:'sprites/hud/armor_full.png',armorHalf:'sprites/hud/armor_half.png',armorEmpty:'sprites/hud/armor_empty.png',
  air:'sprites/hud/air.png',airBursting:'sprites/hud/air_bursting.png',
  xpBg:'sprites/hud/experience_bar_background.png',xpFill:'sprites/hud/experience_bar_progress.png'
 };
 const fallback='https://mcasset.cloud/26.1/assets/minecraft/textures/gui/';
 for(const[k,path]of Object.entries(files)){
  const im=new Image();im.crossOrigin='anonymous';im.onload=()=>{JAVA_GUI[k]=im};
  im.onerror=()=>{const fb=new Image();fb.crossOrigin='anonymous';fb.onload=()=>{JAVA_GUI[k]=fb};fb.onerror=()=>console.warn('Textura Java indisponível:',path);fb.src=fallback+path};
  im.src=JAVA_GUI_BASE+path;
 }
}
loadJavaGui();
async function loadPack(files){
 const mb={},mi={},ma={},gi={},en={};
 for(const f of files){const path='/'+(f.webkitRelativePath||f.name).toLowerCase().replace(/\\/g,'/');if(!/\.png$/.test(path))continue;const name=path.split('/').pop().slice(0,-4);
  if(/\/entity\//.test(path)){if(['zombie','skeleton','creeper','steve'].includes(name)&&(!en[name]||path.includes('/'+name+'/')))en[name]=f}
  for(const k in GUIRX)if(GUIRX[k].test(path))gi[k]=f;
  if(/\/(block|blocks)\//.test(path))mb[name]=f;else if(/\/(item|items)\//.test(path))mi[name]=f;else ma[name]=f;
 }
 let n=0;const over=mb['grass_block_side_overlay']?await createImageBitmap(mb['grass_block_side_overlay']):null;
 const blockAlias={water_still:['water_flow_grey','water_still_grey'],oak_sapling:['sapling_oak','oak_sapling'],spruce_sapling:['sapling_spruce','spruce_sapling'],birch_sapling:['sapling_birch','birch_sapling'],jungle_sapling:['sapling_jungle','jungle_sapling'],acacia_sapling:['sapling_acacia','acacia_sapling'],dark_oak_sapling:['sapling_roofed_oak','sapling_dark_oak','dark_oak_sapling'],mangrove_sapling:['sapling_mangrove','mangrove_propagule'],cherry_sapling:['sapling_cherry','cherry_sapling'],pale_oak_sapling:['sapling_pale_oak','pale_oak_sapling'],poplar_sapling:['sapling_poplar','poplar_sapling'],oak_planks:['planks_oak'],oak_log:['log_oak'],oak_log_top:['log_oak_top'],oak_leaves:['leaves_oak'],spruce_planks:['planks_spruce'],spruce_log:['log_spruce'],spruce_log_top:['log_spruce_top'],spruce_leaves:['leaves_spruce'],birch_planks:['planks_birch'],birch_log:['log_birch'],birch_log_top:['log_birch_top'],birch_leaves:['leaves_birch'],jungle_planks:['planks_jungle'],jungle_log:['log_jungle'],jungle_log_top:['log_jungle_top'],jungle_leaves:['leaves_jungle'],acacia_planks:['planks_acacia'],acacia_log:['log_acacia'],acacia_log_top:['log_acacia_top'],acacia_leaves:['leaves_acacia'],dark_oak_planks:['planks_big_oak','planks_dark_oak'],dark_oak_log:['log_big_oak','log_dark_oak'],dark_oak_log_top:['log_big_oak_top','log_dark_oak_top'],dark_oak_leaves:['leaves_big_oak','leaves_dark_oak']};
 const itemAlias={wooden_pickaxe:['wood_pickaxe'],wooden_axe:['wood_axe'],wooden_shovel:['wood_shovel'],wooden_hoe:['wood_hoe'],wooden_sword:['wood_sword'],golden_pickaxe:['gold_pickaxe'],golden_axe:['gold_axe'],golden_shovel:['gold_shovel'],golden_hoe:['gold_hoe'],golden_sword:['gold_sword'],golden_helmet:['gold_helmet'],golden_chestplate:['gold_chestplate'],golden_leggings:['gold_leggings'],golden_boots:['gold_boots'],golden_apple:['apple_golden'],book:['book_normal'],water_bucket:['bucket_water'],lava_bucket:['bucket_lava'],bucket:['bucket_empty'],bow:['bow_standby'],clock:['clock_item'],cooked_beef:['beef_cooked'],beef:['beef_raw'],cooked_porkchop:['porkchop_cooked'],porkchop:['porkchop_raw'],cooked_chicken:['chicken_cooked'],chicken:['chicken_raw'],cooked_mutton:['mutton_cooked'],mutton:['mutton_raw'],rotten_flesh:['rotten_flesh'],flint_and_steel:['flint_and_steel'],fishing_rod:['fishing_rod_uncast'],boat_dark_oak:['boat_darkoak'],boat_mangrove:['mangrove_boat'],boat_cherry:['cherry_boat'],boat_pale_oak:['pale_oak_boat'],boat_poplar:['poplar_boat'],bamboo_raft:['bamboo_raft'],wooden_door:['door_wood'],iron_door:['door_iron'],redstone:['redstone_dust'],slime_ball:['slimeball']};
 const find=(file,isItem)=>{const m=isItem?mi:mb,aliases=(isItem?itemAlias:blockAlias)[file]||[];for(const nm of[file,...aliases])if(m[nm]||ma[nm])return m[nm]||ma[nm];const norm=file.replace(/^(block_|item_)/,'');for(const[k,v]of Object.entries(m))if(k.replace(/^(block_|item_)/,'')===norm)return v;for(const[k,v]of Object.entries(ma))if(k.replace(/^(block_|item_)/,'')===norm)return v;return null};
 for(const[t,file]of Object.entries(FILES)){const isItem=t.startsWith('i_'),f=find(file,isItem);if(!f)continue;try{const bm=await createImageBitmap(f);paintTile(t,bm,isItem?null:over);if((t=='water'||t=='lava'||t=='portal')&&bm.height>bm.width){PK.add(t);ANIM[t]={bm,n:Math.floor(bm.height/bm.width)}}else if(t=='water'||t=='lava'){PK.add(t);delete ANIM[t]}n++}catch(e){}}
 uploadAtlas();gl.activeTexture(gl.TEXTURE0);let g=0;for(const k in gi){try{GUI[k]=await createImageBitmap(gi[k]);g++}catch(e){}}
 let e2=0;for(const k in en){try{const im=await loadImg(en[k]);if(k=='steve'){if(!skinUser)setSkin(im,false)}else addEntTex(k,im);e2++}catch(e){}}
 for(const c of chunks.values())c.dirty=true;msg(`Pacote carregado: ${n} texturas de blocos/itens, ${g} de interface, ${e2} de entidades.`);if(!n)msg('Nada reconhecido: escolha a pasta textures do resource pack.')}

async function loadVanillaBedrock(){
 msg('Carregando texturas vanilla do Minecraft Bedrock...');const base='https://raw.githubusercontent.com/Mojang/bedrock-samples/main/resource_pack/textures/';
 const blockAlias={water_still:['water_flow_grey','water_still_grey'],water:['water_flow_grey','water_still_grey'],oak_sapling:['sapling_oak','oak_sapling'],spruce_sapling:['sapling_spruce','spruce_sapling'],birch_sapling:['sapling_birch','birch_sapling'],jungle_sapling:['sapling_jungle','jungle_sapling'],acacia_sapling:['sapling_acacia','acacia_sapling'],dark_oak_sapling:['sapling_roofed_oak','sapling_dark_oak','dark_oak_sapling'],mangrove_sapling:['sapling_mangrove','mangrove_propagule'],cherry_sapling:['sapling_cherry','cherry_sapling'],pale_oak_sapling:['sapling_pale_oak','pale_oak_sapling'],poplar_sapling:['sapling_poplar','poplar_sapling'],oak_planks:['planks_oak'],oak_log:['log_oak'],oak_log_top:['log_oak_top'],oak_leaves:['leaves_oak'],spruce_planks:['planks_spruce'],spruce_log:['log_spruce'],spruce_log_top:['log_spruce_top'],spruce_leaves:['leaves_spruce'],birch_planks:['planks_birch'],birch_log:['log_birch'],birch_log_top:['log_birch_top'],birch_leaves:['leaves_birch'],jungle_planks:['planks_jungle'],jungle_log:['log_jungle'],jungle_log_top:['log_jungle_top'],jungle_leaves:['leaves_jungle'],acacia_planks:['planks_acacia'],acacia_log:['log_acacia'],acacia_log_top:['log_acacia_top'],acacia_leaves:['leaves_acacia'],dark_oak_planks:['planks_big_oak','planks_dark_oak'],dark_oak_log:['log_big_oak','log_dark_oak'],dark_oak_log_top:['log_big_oak_top','log_dark_oak_top'],dark_oak_leaves:['leaves_big_oak','leaves_dark_oak']};
 const itemAlias={wooden_pickaxe:['wood_pickaxe'],wooden_axe:['wood_axe'],wooden_shovel:['wood_shovel'],wooden_hoe:['wood_hoe'],wooden_sword:['wood_sword'],golden_pickaxe:['gold_pickaxe'],golden_axe:['gold_axe'],golden_shovel:['gold_shovel'],golden_hoe:['gold_hoe'],golden_sword:['gold_sword'],golden_helmet:['gold_helmet'],golden_chestplate:['gold_chestplate'],golden_leggings:['gold_leggings'],golden_boots:['gold_boots'],golden_apple:['apple_golden'],book:['book_normal'],water_bucket:['bucket_water'],lava_bucket:['bucket_lava'],bucket:['bucket_empty'],bow:['bow_standby'],clock:['clock_item'],cooked_beef:['beef_cooked'],beef:['beef_raw'],cooked_porkchop:['porkchop_cooked'],porkchop:['porkchop_raw'],cooked_chicken:['chicken_cooked'],chicken:['chicken_raw'],cooked_mutton:['mutton_cooked'],mutton:['mutton_raw'],rotten_flesh:['rotten_flesh'],flint_and_steel:['flint_and_steel'],fishing_rod:['fishing_rod_uncast'],boat_oak:['boat_oak'],boat_spruce:['boat_spruce'],boat_birch:['boat_birch'],boat_jungle:['boat_jungle'],boat_acacia:['boat_acacia'],boat_dark_oak:['boat_darkoak'],boat_mangrove:['mangrove_boat'],boat_cherry:['cherry_boat'],boat_pale_oak:['pale_oak_boat'],boat_poplar:['poplar_boat'],bamboo_raft:['bamboo_raft'],wooden_door:['door_wood'],iron_door:['door_iron'],mushroom_stew:['mushroom_stew'],sweet_berries:['sweet_berries'],redstone:['redstone_dust'],lapis_lazuli:['lapis_lazuli'],glowstone_dust:['glowstone_dust'],ender_pearl:['ender_pearl'],slime_ball:['slimeball'],snowball:['snowball'],arrow:['arrow'],iron_ingot:['iron_ingot'],gold_ingot:['gold_ingot'],copper_ingot:['copper_ingot']};
 const tasks=Object.entries(FILES),results=new Array(tasks.length);let cursor=0,loaded=0,failed=0;
 const worker=async()=>{while(true){const ix=cursor++;if(ix>=tasks.length)return;const[t,file]=tasks[ix],folder=t.startsWith('i_')?'items':'blocks',aliases=(t.startsWith('i_')?itemAlias:blockAlias)[file]||[],names=[file,...aliases];let bm=null;
  for(const nm of names){try{const r=await fetch(base+folder+'/'+nm+'.png');if(!r.ok)continue;const blob=await r.blob();bm=await createImageBitmap(blob);break}catch(e){}}
  if(bm){try{paintTile(t,bm,null);if(t=='water'||t=='lava'||t=='portal'){if(bm.height>bm.width){PK.add(t);ANIM[t]={bm,n:Math.floor(bm.height/bm.width)}}else{PK.add(t);delete ANIM[t]}}loaded++}catch(e){failed++}}else failed++;
 }};
 await Promise.all(Array.from({length:7},worker));uploadAtlas();gl.activeTexture(gl.TEXTURE0);for(const c of chunks.values())c.dirty=true;msg('Texturas Bedrock carregadas: '+loaded+' de '+tasks.length+'. '+(failed?'Algumas não existem nessa versão.':'Tudo encontrado!'));
}
function importSkin(){document.exitPointerLock();pickFiles('image/png',false,false,async fs=>{if(!fs[0])return;try{setSkin(await loadImg(fs[0]),true);skinUser=true;msg('Skin importada! Aperte V para ver seu personagem.')}catch(e){msg('Falha ao ler a skin')}})}
function importPack(multi){document.exitPointerLock();msg('Selecione a pasta "textures" do seu Minecraft/resource pack...');pickFiles('image/png',multi,!multi,loadPack)}
 
/* ================= HUD ================= */
const UI={rects:[],scroll:0,held:null,dragStart:null,cont:null,mx:0,my:0,creativeCat:'building',craft2:Array(4).fill(null),craft3:Array(9).fill(null),catScroll:0,offhand:[null]};hud.imageSmoothingEnabled=false;
const sizeS=()=>Math.max(28,Math.min(46,Math.floor(W/20)));
const JAVA_ICON_CACHE=new Map(),JAVA_ICON_BASE='https://raw.githubusercontent.com/InventivetalentDev/minecraft-assets/26.1/assets/minecraft/textures/';
const JAVA_ICON_ALIASES={rotten:['rotten_flesh'],lapis:['lapis_lazuli'],glowdust:['glowstone_dust'],slime:['slime_ball'],stew:['mushroom_stew'],berries:['sweet_berries'],flint_steel:['flint_and_steel'],raft_bamboo:['bamboo_raft'],sapling:['oak_sapling'],clock:['clock_00'],compass:['compass_00'],stonebr:['stone_bricks'],mcobble:['mossy_cobblestone'],wall_cobble:['cobblestone_wall'],hay:['hay_block'],lamp:['redstone_lamp'],door_iron:['iron_door'],boat_oak:['oak_boat'],boat_spruce:['spruce_boat'],boat_birch:['birch_boat'],boat_jungle:['jungle_boat'],boat_acacia:['acacia_boat'],boat_dark_oak:['dark_oak_boat'],boat_mangrove:['mangrove_boat'],boat_cherry:['cherry_boat'],boat_pale_oak:['pale_oak_boat'],door_mangrove:['mangrove_door'],door_cherry:['cherry_door'],door_pale_oak:['pale_oak_door'],door_bamboo:['bamboo_door'],door_crimson:['crimson_door'],door_warped:['warped_door']};
let JAVA_ICON_ACTIVE=0;const JAVA_ICON_QUEUE=[];
function pumpJavaIcons(){while(JAVA_ICON_ACTIVE<4&&JAVA_ICON_QUEUE.length){JAVA_ICON_ACTIVE++;const job=JAVA_ICON_QUEUE.shift();job(()=>{JAVA_ICON_ACTIVE--;pumpJavaIcons()})}}
function javaIcon(id){let e=JAVA_ICON_CACHE.get(id);if(e)return e.im||null;const key=id>=IB?(IT[id]&&IT[id].key):(BKEY[id]||''),names=[key,...(JAVA_ICON_ALIASES[key]||[])];if(key.startsWith('boat_'))names.push(key.slice(5)+'_boat');if(key.startsWith('door_'))names.push(key.slice(5)+'_door');if(key.startsWith('slab_'))names.push(key.slice(5)+'_slab');if(key.startsWith('stair_'))names.push(key.slice(6)+'_stairs');if(key.startsWith('fence_'))names.push(key.slice(6)+'_fence');if(key.startsWith('gate_'))names.push(key.slice(5)+'_fence_gate');if(key.startsWith('trap_'))names.push(key.slice(5)+'_trapdoor');if(key.startsWith('button_'))names.push(key.slice(7)+'_button');if(key.startsWith('plate_'))names.push(key.slice(6)+'_pressure_plate');if(key.startsWith('sign_'))names.push(key.slice(5)+'_sign');if(key.startsWith('hanging_sign_'))names.push(key.slice(13)+'_hanging_sign');if(key.startsWith('wool_'))names.push(key.slice(5)+'_wool');if(key.startsWith('conc_'))names.push(key.slice(5)+'_concrete');const paths=[];for(const n of [...new Set(names)].filter(Boolean)){if(id>=IB)paths.push('item/'+n+'.png');else paths.push('item/'+n+'.png','block/'+n+'.png')};if(id==22)paths.unshift('block/crafting_table.png');e={im:null,loading:true};JAVA_ICON_CACHE.set(id,e);JAVA_ICON_QUEUE.push(done=>{let i=0;const next=()=>{if(i>=paths.length){e.loading=false;done();return}const im=new Image();im.crossOrigin='anonymous';im.onload=()=>{e.im=im;e.loading=false;done()};im.onerror=()=>{i++;next()};im.src=JAVA_ICON_BASE+paths[i++]};next()});pumpJavaIcons();return null}
function drawInventoryFace(tileId,pts,shade,tint){
 const t=Number.isFinite(tileId)?tileId:(TN.white??0),ox=(t&(TPR-1))*16,oy=(t/TPR|0)*16;
 const p0=pts[0],p1=pts[1],p3=pts[3];
 hud.save();hud.imageSmoothingEnabled=false;hud.beginPath();hud.moveTo(p0[0],p0[1]);for(let i=1;i<4;i++)hud.lineTo(pts[i][0],pts[i][1]);hud.closePath();hud.clip();
 // Mapeia os 16x16 pixels diretamente nos dois vetores da face, sem esticar pelo bounding box.
 hud.save();hud.transform((p1[0]-p0[0])/16,(p1[1]-p0[1])/16,(p3[0]-p0[0])/16,(p3[1]-p0[1])/16,p0[0],p0[1]);
 hud.drawImage(AT,ox,oy,16,16,0,0,16,16);hud.restore();
 const minX=Math.min(...pts.map(p=>p[0])),maxX=Math.max(...pts.map(p=>p[0])),minY=Math.min(...pts.map(p=>p[1])),maxY=Math.max(...pts.map(p=>p[1]));
 if(shade<1){hud.fillStyle=`rgba(0,0,0,${1-shade})`;hud.fillRect(minX,minY,maxX-minX,maxY-minY)}
 if(tint){hud.globalCompositeOperation='multiply';hud.fillStyle=`rgb(${Math.max(0,Math.min(255,tint[0]*255|0))},${Math.max(0,Math.min(255,tint[1]*255|0))},${Math.max(0,Math.min(255,tint[2]*255|0))})`;hud.fillRect(minX,minY,maxX-minX,maxY-minY)}
 hud.restore();
}
function inventoryBlockModel(id,x,y,s){
 const b=BL[id];if(!b||b.cross||b.liq)return false;
 if(!(b.s||b.box||b.fence||b.gate))return false;
 const key=BKEY[id]||'',meta=b.cont=='chest'?1:0,tint=b.leaf?(b.tc||[.45,.72,.3]):null;
 const project=q=>[x+s*.50+(q[0]-q[2])*s*.285,y+s*.16+(1-q[1])*s*.44+(q[0]+q[2])*s*.175];
 let boxes;
 if(key.startsWith('fence_')){boxes=[[.375,0,.375,.625,1,.625],[.625,.375,.4375,1,.9375,.5625],[0,.375,.4375,.375,.9375,.5625],[.4375,.375,.625,.5625,.9375,1],[.4375,.375,0,.5625,.9375,.375]]}
 else if(key.startsWith('wall_')){boxes=[[.25,0,.25,.75,1,.75],[.75,0,.3125,1,.8125,.6875],[0,0,.3125,.25,.8125,.6875],[.3125,0,.75,.6875,.8125,1],[.3125,0,0,.6875,.8125,.25]]}
 else if(key==='glass_pane'||key==='iron_bars'){boxes=[[.4375,0,0,.5625,1,1],[0,0,.4375,1,1,.5625]]}
 else boxes=b.box?(typeof b.box==='function'?b.box(meta,0,0,0):b.box):[[0,0,0,1,1,1]];
 if(!Array.isArray(boxes)||!boxes.length)boxes=[[0,0,0,1,1,1]];
 boxes=[...boxes].sort((a,b)=>(a[0]+a[2]+a[3]+a[5])-(b[0]+b[2]+b[3]+b[5]));
 const tileFor=(f,q)=>{
  if(q[6]!==undefined)return q[6];
  if(b.faceTex)return b.faceTex(meta,f,0,0,0);
  if(b.log){const axis=meta&3,end=axis==0?(f==2||f==3):axis==1?(f==0||f==1):(f==4||f==5);return end?(b.t[0]??b.t[2]):(b.t[2]??b.t[0])}
  if(f==2)return b.ttop?b.ttop(meta):(b.t[0]??b.t[2]);
  if(f==3)return b.t[1]??b.t[2]??b.t[0];
  return b.tmeta?b.tmeta(meta):(b.t[2]??b.t[0]);
 };
 for(const q of boxes){
  const x0=q[0],y0=q[1],z0=q[2],x1=q[3],y1=q[4],z1=q[5];
  // Top, left and right faces: proper parallelogram texture projection.
  const top=[[x0,y1,z0],[x1,y1,z0],[x1,y1,z1],[x0,y1,z1]].map(project);
  const left=[[x0,y1,z1],[x0,y1,z0],[x0,y0,z0],[x0,y0,z1]].map(project);
  const right=[[x0,y1,z1],[x1,y1,z1],[x1,y0,z1],[x0,y0,z1]].map(project);
  drawInventoryFace(tileFor(2,q),top,1,tint);
  drawInventoryFace(tileFor(1,q),left,.70,tint);
  drawInventoryFace(tileFor(4,q),right,.86,tint);
 }
 return true;
}
function icon(id,x,y,s){
 hud.imageSmoothingEnabled=false;hud.save();
 if(id<IB&&inventoryBlockModel(id,x+s*.04,y+s*.04,s*.92)){hud.restore();return}
 const ji=javaIcon(id);
 if(ji){hud.drawImage(ji,x+Math.max(0,s*.06),y+Math.max(0,s*.06),s*.88,s*.88);hud.restore();return}
 const tileId=id>=IB?(IT[id]&&IT[id].tile):(()=>{const b=BL[id];if(!b)return null;if(b.cross&&b.saplingType)return TN['sapling_'+b.saplingType]??TN.sapling;if(b.log)return b.t[0];return b.t[2]??b.t[0]})();
 if(tileId!=null&&Number.isFinite(tileId)){const ox=(tileId&(TPR-1))*16,oy=(tileId/TPR|0)*16;hud.drawImage(AT,ox,oy,16,16,x+Math.max(0,s*.06),y+Math.max(0,s*.06),s*.88,s*.88)}
 if(id<IB&&BL[id]&&BL[id].leaf){const tint=BL[id].tc||[.45,.72,.3];hud.globalCompositeOperation='multiply';hud.fillStyle=`rgb(${tint[0]*255|0},${tint[1]*255|0},${tint[2]*255|0})`;hud.fillRect(x+s*.06,y+s*.06,s*.88,s*.88)}
 hud.restore();
}

function txt(s,x,y,fs,col='#fff',al='left'){hud.font=`bold ${fs}px monospace`;hud.textAlign=al;hud.fillStyle='#000';hud.fillText(s,x+1,y+1);hud.fillStyle=col;hud.fillText(s,x,y)}
function slot(i,x,y,sz,hl,bare){if(!bare){hud.fillStyle=hl?'rgba(255,255,255,.35)':'rgba(30,30,30,.7)';hud.fillRect(x,y,sz,sz);hud.strokeStyle=hl?'#fff':'#666';hud.lineWidth=hl?3:1;hud.strokeRect(x+.5,y+.5,sz-1,sz-1)}else if(hl){hud.fillStyle='rgba(255,255,255,.4)';hud.fillRect(x,y,sz,sz)}
 const s=inv[i];if(s){icon(s.id,x+(bare?0:4),y+(bare?0:4),sz-(bare?0:8));if(s.n>1&&gm!=1)txt(''+s.n,x+sz-3,y+sz-4,Math.max(10,Math.floor(sz/3)),'#fff','right')}}
// ícones de status (usam o pacote se existir)
function stIcon(kind,state,x,y,s){const g=hud;g.imageSmoothingEnabled=false;
 const pick=kind==='h'?(state===2?JAVA_GUI.heartFull:state===1?JAVA_GUI.heartHalf:JAVA_GUI.heartEmpty):kind==='f'?(state===2?JAVA_GUI.foodFull:state===1?JAVA_GUI.foodHalf:JAVA_GUI.foodEmpty):(state===2?JAVA_GUI.armorFull:state===1?JAVA_GUI.armorHalf:JAVA_GUI.armorEmpty);
 const base=kind==='h'?JAVA_GUI.heartEmpty:kind==='f'?JAVA_GUI.foodEmpty:JAVA_GUI.armorEmpty;
 if(base&&(state===0||pick)){g.drawImage(base,x,y,s,s);if(state>0&&pick&&pick!==base)g.drawImage(pick,x,y,s,s);return;}
 const L=GUI.ic,sp={h:{e:'hc',h:'hh',f:'hf'},f:{e:'fe',h:'fh',f:'ff'},a:{e:'ae',h:'ah',f:'af'}}[kind];
 if(GUI[sp.f]){const im=GUI[state==2?sp.f:state==1?sp.h:sp.e]||GUI[sp.f];if(kind==='h'&&GUI.hc)g.drawImage(GUI.hc,x,y,s,s);if(state>0||kind!=='h')g.drawImage(im,x,y,s,s);return}
 if(L){const o={h:[[16,0],[61,0],[52,0]],f:[[16,27],[61,27],[52,27]],a:[[16,9],[25,9],[34,9]]}[kind];g.drawImage(L,o[0][0],o[0][1],9,9,x,y,s,s);if(state>0)g.drawImage(L,o[state][0],o[state][1],9,9,x,y,s,s);return}
 g.fillStyle=kind==='h'?(state===2?'#e22':state===1?'#e92':'#333'):kind==='f'?(state===2?'#d82':state===1?'#a62':'#333'):(state===2?'#ccd':state===1?'#889':'#333');g.fillRect(x,y,s*.8,s*.8)}
function hotG(){const sz=sizeS();if(JAVA_GUI.hb||GUI.wg){const sc=sz/20,x0=W/2-91*sc,y0=HH-22*sc-6;return{gui:1,sc,x0,y0,w:182*sc,rx:i=>({x:x0+(3+20*i)*sc,y:y0+3*sc,w:16*sc,h:16*sc})}}
 const x0=Math.floor((W-9*sz)/2),y0=HH-sz-8;return{gui:0,sz,x0,y0,w:9*sz,rx:i=>({x:x0+i*sz,y:y0,w:sz,h:sz})}}
function drawHUD(now){
 const g=hud;g.clearRect(0,0,W,HH);g.imageSmoothingEnabled=false;UI.rects=[];UI.hover=-1;
 const sz=sizeS(),hg=hotG(),y0=hg.gui?hg.y0:hg.y0,x0=hg.x0;
 if(flash>0){g.fillStyle=`rgba(200,0,0,${Math.min(.5,flash)})`;g.fillRect(0,0,W,HH)}
 if(blockAt(P.x,P.y+1.62,P.z)==10){g.fillStyle='rgba(20,60,160,.3)';g.fillRect(0,0,W,HH)}
 drawWeather(g);
 if(!invOpen&&!dead&&gm!=2){if(JAVA_GUI.cross)g.drawImage(JAVA_GUI.cross,W/2-7.5,HH/2-7.5,15,15);else{g.fillStyle='#fff';g.fillRect(W/2-1,HH/2-9,2,18);g.fillRect(W/2-9,HH/2-1,18,2);}
  if(P.mine&&P.mine.p>0){g.fillStyle='rgba(0,0,0,.5)';g.fillRect(W/2-30,HH/2+20,60,6);g.fillStyle='#fff';g.fillRect(W/2-30,HH/2+20,60*Math.min(1,P.mine.p),6)}}
 const hs_=inv[sel];if(hs_&&!invOpen&&!dead&&gm!=2&&view==0){g.save();g.translate(W-sz*3,HH-sz*2.4);g.rotate(-.3+Math.sin(swing)*.5);icon(hs_.id,-sz*.8,-sz*.8,sz*2.2);g.restore()}
 if(gm!=2&&!invOpen){
  if(hg.gui){const im=JAVA_GUI.hb||GUI.wg;if(JAVA_GUI.hb)g.drawImage(JAVA_GUI.hb,hg.x0,hg.y0,hg.w,22*hg.sc);else g.drawImage(GUI.wg,0,0,182,22,hg.x0,hg.y0,hg.w,22*hg.sc);
   const sx=Math.round(hg.x0+(1+20*sel)*hg.sc),sy=Math.round(hg.y0-hg.sc);if(JAVA_GUI.hs)g.drawImage(JAVA_GUI.hs,sx,sy,24*hg.sc,24*hg.sc);else if(GUI.wg)g.drawImage(GUI.wg,0,22,24,24,sx,sy,24*hg.sc,24*hg.sc)}
  for(let i=0;i<9;i++){const r=hg.rx(i);if(hg.gui){const s=inv[i];if(s){icon(s.id,r.x,r.y,r.w);if(s.n>1&&gm!=1)txt(''+s.n,r.x+r.w,r.y+r.h,Math.max(10,r.w/2.2|0),'#fff','right')}}else slot(i,r.x,r.y,r.w,i==sel);
   const ii=i;UI.rects.push({x:r.x,y:r.y,w:r.w,h:r.h,fn:()=>{sel=ii}})}}
 const top=y0-4,s9=sz*.4;
 if(gm==0&&!invOpen){
  for(let i=0;i<10;i++)stIcon('h',P.hp>=i*2+2?2:P.hp>=i*2+1?1:0,x0+i*s9*1.02,top-s9,s9);
  for(let i=0;i<10;i++)stIcon('f',P.food>=i*2+2?2:P.food>=i*2+1?1:0,x0+hg.w-(i+1)*s9*1.02,top-s9,s9);
  const ap=armorPts();if(ap>0)for(let i=0;i<10;i++)stIcon('a',ap>=i*2+2?2:ap>=i*2+1?1:0,x0+i*s9*1.02,top-s9*2.1,s9);
  if(P.air<9.9)for(let i=0;i<10;i++){const im=i<Math.ceil(P.air)?JAVA_GUI.air:JAVA_GUI.airBursting;if(im)g.drawImage(im,x0+hg.w-(i+1)*s9*1.02,top-s9*2.1,s9,s9);else if(i<Math.ceil(P.air)){g.fillStyle='#8cf';g.beginPath();g.arc(x0+hg.w-(i+1)*s9*1.02+s9*.5,top-s9*1.6,s9*.35,0,7);g.fill()}}}
 hudExtra(g,hg);
 if(hs_&&!invOpen&&gm!=2)txt(nameOf(hs_.id),W/2,top-(gm==0?s9*2.6:s9*.6),Math.floor(sz*.4),'#fff','center');
 let y=20;for(const m of msgs){const a=Math.max(0,Math.min(1,(7000-(now-m.t))/1500));if(a>0||invOpen)txt(m.s,8,y,15,`rgba(255,255,255,${invOpen?1:a})`);y+=18}
 if(showDbg){const c=chunkAt(P.x,P.z),b=c?BIOMES[c.bio[(Math.floor(P.x)&15)+(Math.floor(P.z)&15)*16]]:'-';
  txt(`FPS ${fps|0}  XYZ ${P.x.toFixed(1)} ${P.y.toFixed(1)} ${P.z.toFixed(1)}  Bioma: ${b}  Chunks ${chunks.size}  Mobs ${mobs.length}  Armadura ${armorPts()}  Hora ${(tod*24).toFixed(1)}  Seed ${SEED}`,8,HH-26,13,'#ff8')}
 if(invOpen)drawInv();
 if(dead){g.fillStyle='rgba(120,0,0,.6)';g.fillRect(0,0,W,HH);txt('VOCÊ MORREU',W/2,HH/2-10,48,'#fff','center');txt('Pressione R ou clique para renascer',W/2,HH/2+30,20,'#fff','center');UI.rects.push({x:0,y:0,w:W,h:HH,fn:respawn})}
 else if(!locked&&!invOpen){g.fillStyle='rgba(0,0,0,.55)';g.fillRect(0,0,W,HH);
  txt('MineJS',W/2,HH/2-110,56,'#7d7','center');txt(spawned?'Clique para jogar':'Gerando mundo...',W/2,HH/2-60,26,'#fff','center');
  const L=['WASD mover · Espaço pular (2x = voar no criativo) · Shift descer/agachar · Ctrl correr','Botão esquerdo quebrar/atacar · Direito colocar/comer/vestir armadura · 1-9 ou roda: itens','E inventário · G modo de jogo · V câmera 3ª pessoa · Q soltar item · F3 debug','K importar SKIN (PNG 64x32/64x64) · O carregar PASTA DE TEXTURAS · /pack bedrock baixa texturas oficiais','T chat/comandos (Tab completa · ↑ histórico) · R renascer · Placas: clique direito para editar'];
  L.forEach((s,i)=>txt(s,W/2,HH/2+i*24,15,'#ddd','center'))}
}
/* ================= COMANDOS ================= */
function findItem(q){if(/^\d+$/.test(q)){const n=+q;return(BL[n]&&n>0)||IT[n]?n:null}const nz=s=>s.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'');q=nz(q);
 for(const id of ALLIDS)if(nz(nameOf(id)).includes(q))return id;return null}
function setGM(n){gm=n;P.fly=n==2;P.mine=null;if(n==2){invOpen=false;UI.cont=null;UI.held=null;}msg('Modo: '+['sobrevivência','criativo','espectador'][n]);if(n==0&&!inv.some(s=>s))addInv(I.wooden_pickaxe,1)}
/* ================= CHAT JAVA-STYLE ================= */
const chatBox=document.createElement('div');chatBox.style.cssText='position:fixed;left:18px;bottom:72px;z-index:2147483647;width:min(620px,80vw);display:none;font:14px monospace;color:#fff;text-shadow:1px 1px #222;pointer-events:auto';
const chatLog=document.createElement('div');chatLog.style.cssText='max-height:42vh;overflow:hidden;padding:8px;background:rgba(0,0,0,.48);white-space:pre-wrap;word-break:break-word';
const chatInput=document.createElement('input');chatInput.type='text';chatInput.autocomplete='off';chatInput.spellcheck=false;chatInput.placeholder='Mensagem ou comando';chatInput.style.cssText='box-sizing:border-box;width:100%;margin-top:4px;padding:8px;background:rgba(0,0,0,.78);border:1px solid #aaa;color:white;font:15px monospace;outline:none';
const chatSuggest=document.createElement('div');chatSuggest.style.cssText='display:none;background:rgba(0,0,0,.8);padding:6px;max-height:120px;overflow:auto';chatBox.append(chatLog,chatSuggest,chatInput);document.body.append(chatBox);
let chatOpen=false,chatHistory=[],chatHistPos=0;
const chatCommands=['help','gamemode','give','tp','time','weather','difficulty','gamerule','effect','xp','setblock','fill','clone','summon','kill','heal','feed','clear','dimension','locate','biome','spawnpoint','fly','speed','fov','sens','rd','save','load','skin','pack','view','armor','say','me','list','seed','playsound','tell','msg'];
function refreshChat(){chatLog.innerHTML='';for(const m of msgs.slice(-10)){const d=document.createElement('div');d.textContent=m.s;chatLog.append(d)}chatLog.scrollTop=chatLog.scrollHeight}
function openChat(pref=''){chatOpen=true;chatBox.style.display='block';chatInput.value=pref;chatInput.focus();chatInput.setSelectionRange(chatInput.value.length,chatInput.value.length);document.exitPointerLock();refreshChat();updateChatSuggest()}
function closeChat(){chatOpen=false;chatBox.style.display='none';chatSuggest.style.display='none';for(const k in keys)keys[k]=false}
function updateChatSuggest(){const v=chatInput.value.trim(),q=v.startsWith('/')?v.slice(1).split(/\s/)[0].toLowerCase():'';const hits=q?chatCommands.filter(x=>x.startsWith(q)).slice(0,8):[];chatSuggest.innerHTML='';for(const h of hits){const d=document.createElement('div');d.textContent='/'+h;chatSuggest.append(d)}chatSuggest.style.display=hits.length?'block':'none'}
chatInput.addEventListener('input',updateChatSuggest);chatInput.addEventListener('keydown',e=>{e.stopPropagation();if(e.key==='Enter'){const v=chatInput.value.trim();if(v){chatHistory.push(v);chatHistPos=chatHistory.length;cmd(v);refreshChat()}closeChat();hc.requestPointerLock();e.preventDefault()}else if(e.key==='Escape'){closeChat();hc.requestPointerLock();e.preventDefault()}else if(e.key==='ArrowUp'){chatHistPos=Math.max(0,chatHistPos-1);chatInput.value=chatHistory[chatHistPos]||'';updateChatSuggest();e.preventDefault()}else if(e.key==='ArrowDown'){chatHistPos=Math.min(chatHistory.length,chatHistPos+1);chatInput.value=chatHistory[chatHistPos]||'';updateChatSuggest();e.preventDefault()}else if(e.key==='Tab'){const v=chatInput.value.trim(),q=v.startsWith('/')?v.slice(1).split(/\s/)[0].toLowerCase():'';const h=chatCommands.find(x=>x.startsWith(q));if(h){chatInput.value='/'+h+' ';updateChatSuggest()}e.preventDefault()}});
/* ================= INPUT ================= */
const GAMEKEYS=new Set(['KeyW','KeyA','KeyS','KeyD','Space','ShiftLeft','ControlLeft','KeyE','KeyG','KeyQ','KeyT','KeyR','KeyK','KeyO','KeyV','F3','Slash',...'123456789'.split('').map(i=>'Digit'+i)]);
on(document,'keydown',e=>{
 if(chatOpen){e.stopPropagation();return}
 if(!locked&&!invOpen&&!dead&&e.code!='F3')return;
 if(GAMEKEYS.has(e.code)){e.preventDefault();e.stopPropagation()}
 if(e.repeat)return;keys[e.code]=true;const now=performance.now();
 if(e.code=='KeyR'&&dead)respawn();
 if(dead)return;
 if(e.code.startsWith('Digit'))sel=+e.code[5]-1;
 if(e.code=='KeyE'&&gm!=2){invOpen=!invOpen;UI.held=null;if(invOpen){UI.cont=null;UI.scroll=0;document.exitPointerLock()}else{if(UI.cont&&UI.cont.t=='crafting')UI.craft3=UI.cont.s;UI.cont=null;hc.requestPointerLock()}}
 if(e.code=='KeyG')setGM((gm+1)%3);
 if(e.code=='KeyV')view=(view+1)%3;
 if(e.code=='KeyK')importSkin();
 if(e.code=='KeyO')importPack(false);
 if(e.code=='F3')showDbg=!showDbg;
 if(e.code=='KeyQ'&&inv[sel]&&gm!=2){const s=inv[sel];spawnItem(P.x-Math.sin(P.yaw)*.8,P.y+1.4,P.z-Math.cos(P.yaw)*.8,s.id,1);items[items.length-1].age=-1;if(gm==0){s.n--;if(!s.n)inv[sel]=null}}
 if(e.code=='Space'){if(gm==1&&now-(P.lastSp||0)<300)P.fly=!P.fly;P.lastSp=now}
 if(e.code=='KeyW'){if(now-(P.lastW||0)<300)P.spr=true;P.lastW=now}
 if(e.code=='KeyT'||e.code=='Slash'){openChat(e.code=='Slash'?'/':'')}
},true);
on(document,'keyup',e=>{keys[e.code]=false;if(e.code=='KeyW')P.spr=false},true);
on(document,'mousemove',e=>{UI.mx=e.clientX;UI.my=e.clientY;if(locked){P.yaw-=e.movementX*SENS;P.pitch=Math.max(-1.55,Math.min(1.55,P.pitch-e.movementY*SENS))}});
on(hc,'mousedown',e=>{e.preventDefault();
 
 if(invOpen||dead){let hit=false;for(let i=UI.rects.length-1;i>=0;i--){const r=UI.rects[i];if(e.clientX>=r.x&&e.clientX<=r.x+r.w&&e.clientY>=r.y&&e.clientY<=r.y+r.h){r.fn();hit=true;break}}
  if(!hit&&UI.cont&&UI.cont.t!='crafting')closeContainer();return}
 if(!locked){if(spawned)hc.requestPointerLock();return}
 if(e.button==0)mouse.l=true;if(e.button==2){mouse.r=true;P.placeCd=0}});
on(document,'mouseup',e=>{if(e.button==0){mouse.l=false;P.mine=null;
 if(invOpen&&UI.held&&UI.dragStart){const dx=e.clientX-UI.dragStart.x,dy=e.clientY-UI.dragStart.y,drag=Math.hypot(dx,dy)>9,inside=UI.rects.some(r=>e.clientX>=r.x&&e.clientX<=r.x+r.w&&e.clientY>=r.y&&e.clientY<=r.y+r.h);if(drag&&!inside)dropHeld()}
 }if(e.button==2)mouse.r=false});
on(hc,'contextmenu',e=>e.preventDefault());
on(hc,'wheel',e=>{e.preventDefault();if(invOpen){if(gm==1){const a=CATIDS[UI.creativeCat]||[],cols=Math.max(5,Math.min(9,Math.floor((W-Math.max(145,Math.min(205,W*.17))-220)/sizeS()))),rows=Math.max(4,Math.floor((HH-HH*.12-90)/sizeS())),mx=Math.max(0,Math.ceil(a.length/cols)-rows);UI.scroll=Math.max(0,Math.min(mx,UI.scroll+Math.sign(e.deltaY)))}}else sel=(sel+Math.sign(e.deltaY)+9)%9},{passive:false});
on(document,'pointerlockchange',()=>{locked=document.pointerLockElement===hc;if(!locked)mouse.l=mouse.r=false});
on(window,'blur',()=>{for(const k in keys)keys[k]=false});
 
/* ================= TEXTO 3D DAS PLACAS ================= */
const SIGN_FONT={
'A':['01110','10001','10001','11111','10001','10001','10001'],'B':['11110','10001','10001','11110','10001','10001','11110'],'C':['01111','10000','10000','10000','10000','10000','01111'],'D':['11110','10001','10001','10001','10001','10001','11110'],'E':['11111','10000','10000','11110','10000','10000','11111'],'F':['11111','10000','10000','11110','10000','10000','10000'],'G':['01111','10000','10000','10111','10001','10001','01111'],'H':['10001','10001','10001','11111','10001','10001','10001'],'I':['11111','00100','00100','00100','00100','00100','11111'],'J':['00111','00010','00010','00010','10010','10010','01100'],'K':['10001','10010','10100','11000','10100','10010','10001'],'L':['10000','10000','10000','10000','10000','10000','11111'],'M':['10001','11011','10101','10101','10001','10001','10001'],'N':['10001','11001','10101','10011','10001','10001','10001'],'O':['01110','10001','10001','10001','10001','10001','01110'],'P':['11110','10001','10001','11110','10000','10000','10000'],'Q':['01110','10001','10001','10001','10101','10010','01101'],'R':['11110','10001','10001','11110','10100','10010','10001'],'S':['01111','10000','10000','01110','00001','00001','11110'],'T':['11111','00100','00100','00100','00100','00100','00100'],'U':['10001','10001','10001','10001','10001','10001','01110'],'V':['10001','10001','10001','10001','10001','01010','00100'],'W':['10001','10001','10001','10101','10101','10101','01010'],'X':['10001','10001','01010','00100','01010','10001','10001'],'Y':['10001','10001','01010','00100','00100','00100','00100'],'Z':['11111','00001','00010','00100','01000','10000','11111'],
'0':['01110','10001','10011','10101','11001','10001','01110'],'1':['00100','01100','00100','00100','00100','00100','01110'],'2':['01110','10001','00001','00010','00100','01000','11111'],'3':['11110','00001','00001','01110','00001','00001','11110'],'4':['00010','00110','01010','10010','11111','00010','00010'],'5':['11111','10000','10000','11110','00001','00001','11110'],'6':['01110','10000','10000','11110','10001','10001','01110'],'7':['11111','00001','00010','00100','01000','01000','01000'],'8':['01110','10001','10001','01110','10001','10001','01110'],'9':['01110','10001','10001','01111','00001','00001','01110'],
'.':['00000','00000','00000','00000','00000','00110','00110'],',':['00000','00000','00000','00000','00110','00110','00100'],':':['00000','00110','00110','00000','00110','00110','00000'],'!':['00100','00100','00100','00100','00100','00000','00100'],'?':['01110','10001','00001','00010','00100','00000','00100'],'-':['00000','00000','00000','11111','00000','00000','00000'],'+':['00000','00100','00100','11111','00100','00100','00000'],'/':['00001','00010','00010','00100','01000','01000','10000'],' ':['00000','00000','00000','00000','00000','00000','00000']};
function drawSignText3D(camera){
 if(!signTexts.size)return;
 const white=TN.white??0,uu=tu(white,.5),vv=tv(white,.5),pw=.0082,ph=.0082,stepX=pw*1.18,stepY=ph*1.22;
 const quad=(a,b,c,d)=>{const base=DN;for(const q of[a,b,c,d])DV.push(q[0],q[1],q[2],uu,vv,.12,.085,.045,1,0);DI.push(base,base+1,base+2,base,base+2,base+3);DN+=4};
 for(const[key,lines]of signTexts){if(!lines||!lines.some(Boolean))continue;const q=key.split(',').map(Number);if(q.length!==3)continue;const[x,y,z]=q,id=getBlock(x,y,z),b=BL[id];if(!b||!b.sign||Math.hypot(x+.5-camera[0],y+.6-camera[1],z+.5-camera[2])>22)continue;
  const mt=getMeta(x,y,z),vertical=(mt&3)<2;
  for(let side=0;side<2;side++)for(let row=0;row<4;row++){
   const str=String(lines[row]||'').toUpperCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'').slice(0,15),total=str.length*6-1,startU=(1-total*stepX)/2;
   for(let ci=0;ci<str.length;ci++){const glyph=SIGN_FONT[str[ci]]||SIGN_FONT['?'];for(let gy=0;gy<7;gy++)for(let gx=0;gx<5;gx++)if(glyph[gy][gx]=='1'){
    const u=startU+(ci*6+gx)*stepX,v0=.42-row*8*stepY-gy*stepY;
    const u0=u,u1=u+pw,v1=v0-ph;
    if(vertical){const xx=x+(side?.5625:.4375)+(side?.002:-.002),z0=side?z+.94-(u1):z+.06+u0,z1=side?z+.94-u0:z+.06+u1;quad([xx,y+.39+v1,z0],[xx,y+.39+v1,z1],[xx,y+.39+v0,z1],[xx,y+.39+v0,z0]);}
    else{const zz=z+(side?.5625:.4375)+(side?.002:-.002),x0=side?x+.94-u1:x+.06+u0,x1=side?x+.94-u0:x+.06+u1;quad([x0,y+.39+v1,zz],[x1,y+.39+v1,zz],[x1,y+.39+v0,zz],[x0,y+.39+v0,zz]);}
   }}
  }
 }
 if(DI.length)dflush();
}

/* ================= RENDER ================= */
const SKYN=[.02,.03,.09],SKYD=[.53,.74,.93];
function render(now){
 const eyeY=P.y+1.62,e=[P.x,eyeY,P.z],cp=Math.cos(P.pitch),f=[-Math.sin(P.yaw)*cp,Math.sin(P.pitch),-Math.cos(P.yaw)*cp];
 let cam=e,cdir=f;
 if(view>0&&gm!=2){const s=view==1?-1:1,r=ray(e[0],e[1],e[2],f[0]*s,f[1]*s,f[2]*s,4.2,true),d=r?Math.max(.4,r.t-.3):4.2;cam=[e[0]+f[0]*s*d,e[1]+f[1]*s*d,e[2]+f[2]*s*d];cdir=view==1?f:[-f[0],-f[1],-f[2]]}
 const sa=Math.sin(tod*6.2832);dayF=dim==0?sm(Math.max(0,Math.min(1,(sa+.12)/.42))):(dim==1?.55:.35);if(dim==0&&weather)dayF*=weather==2?.55:.75;
 let sk=[0,1,2].map(i=>SKYN[i]+(SKYD[i]-SKYN[i])*dayF);
 const ss=Math.max(0,1-Math.abs(sa)/.3)*.5;if(ss>0)sk=[lp(sk[0],.95,ss),lp(sk[1],.5,ss),lp(sk[2],.3,ss)];
 const under=blockAt(cam[0],cam[1],cam[2])==10,lava=blockAt(cam[0],cam[1],cam[2])==26;
 let fogc=sk,fn=RD*16*.55,ff=RD*16-6;
 if(dim==1){fogc=[.22,.04,.03];fn=6;ff=RD*16*.7}else if(dim==2){fogc=[.07,.03,.12]}
 if(weather&&!dim){fogc=fogc.map(v=>v*(weather==2?.5:.7));ff*=.75}
 if(lightning>0&&!dim)fogc=fogc.map(v=>Math.min(1,v+lightning*.6));
 if(under){const q=.3+dayF*.7;fogc=[.08*q,.2*q,.5*q];fn=0;ff=22}if(lava){fogc=[.8,.25,.05];fn=0;ff=3}
 gl.clearColor(fogc[0],fogc[1],fogc[2],1);gl.clear(gl.COLOR_BUFFER_BIT|gl.DEPTH_BUFFER_BIT);
 const asp=gc.width/gc.height,fov=(FOVB+(P.sprint?10:0))*Math.PI/180,M=mul(persp(fov,asp,.08,600),look(cam,cdir));
gl.useProgram(prog);
bindUnit(0,tex);
gl.uniformMatrix4fv(U.M,false,new Float32Array(M));
gl.uniform3f(U.eye,cam[0],cam[1],cam[2]);

const skyFactor=P.fx.nv?1:(dim==0?dayF:0);
const sunrise=dim==0?Math.max(0,1-Math.abs(sa)/.30):0;
const rain=dim==0&&weather?(weather==2?1:.65):0;

gl.uniform1f(U.day,skyFactor);
gl.uniform1f(U.amb,P.fx.nv?.72:(dim==1?.25:.05));
gl.uniform1f(U.sunrise,sunrise);
gl.uniform1f(U.rain,rain);
gl.uniform3f(U.fog,fogc[0],fogc[1],fogc[2]);
 gl.disable(gl.DEPTH_TEST);gl.uniform1f(U.fn,1e8);gl.uniform1f(U.ff,2e8);
 if(!under&&!lava&&dim==0)for(const[sg_,c]of[[1,[1,.95,.6]],[-1,[.85,.9,1]]]){let dx=Math.cos(tod*6.2832)*sg_,dy=sa*sg_,dz=.3*sg_;const l=Math.hypot(dx,dy,dz);dx/=l;dy/=l;dz/=l;
  if(dy<-.15)continue;let rx=dz,rz=-dx;const rl=Math.hypot(rx,rz)||1;rx/=rl;rz/=rl;const ux=-dy*rz,uy=dx*rz-dz*rx,uz=dy*rx,ul=Math.hypot(ux,uy,uz)||1;
  const cx=cam[0]+dx*100,cy=cam[1]+dy*100,cz=cam[2]+dz*100,S=sg_>0?9:6;
  for(let k=0;k<4;k++){const a=[[-1,-1],[1,-1],[1,1],[-1,1]][k];DV.push(cx+rx*S*a[0]+ux/ul*S*a[1],cy+uy/ul*S*a[1],cz+rz*S*a[0]+uz/ul*S*a[1],UVC[k][0]*.01,UVC[k][1]*.01,c[0],c[1],c[2],1,1)}
  DI.push(DN,DN+1,DN+2,DN,DN+2,DN+3);DN+=4}
 dflush();
 gl.enable(gl.DEPTH_TEST);gl.uniform1f(U.fn,fn);gl.uniform1f(U.ff,ff);
 const vis=[];for(const c of chunks.values()){if(!c.mo&&!c.mt)continue;const cx=c.cx*16+8-cam[0],cz=c.cz*16+8-cam[2],d=Math.hypot(cx,cz);if(d>RD*16+10)continue;
  if(d>24&&cx*cdir[0]+cz*cdir[2]<-d*.45)continue;c.dd=d;vis.push(c)}
 for(const c of vis)if(c.mo){gl.bindVertexArray(c.mo.vao);gl.drawElements(gl.TRIANGLES,c.mo.n,gl.UNSIGNED_INT,0)}
 drawEntities();
 const t=P.target;
 if(t&&gm!=2){const b=BL[t.id],mt=getMeta(t.x,t.y,t.z),boxes=b.box?(typeof b.box=='function'?b.box(mt,t.x,t.y,t.z):b.box):[[0,0,0,1,1,1]],o=.003;let base=0;
  for(const q of boxes||[]){const x0=t.x+q[0]-o,y0=t.y+q[1]-o,z0=t.z+q[2]-o,x1=t.x+q[3]+o,y1=t.y+q[4]+o,z1=t.z+q[5]+o;
   const v=[[x0,y0,z0],[x1,y0,z0],[x1,y0,z1],[x0,y0,z1],[x0,y1,z0],[x1,y1,z0],[x1,y1,z1],[x0,y1,z1]];
   for(const p of v)DV.push(p[0],p[1],p[2],.001,.001,0,0,0,1,1);
   const ix=[0,1,1,2,2,3,3,0,4,5,5,6,6,7,7,4,0,4,1,5,2,6,3,7];for(const n of ix)DI.push(base+n);base+=8;DN+=8}
  dflush(gl.LINES)}
 gl.enable(gl.BLEND);gl.depthMask(false);vis.sort((a,b)=>b.dd-a.dd);
 for(const c of vis)if(c.mt){gl.bindVertexArray(c.mt.vao);gl.drawElements(gl.TRIANGLES,c.mt.n,gl.UNSIGNED_INT,0)}
 if(P.mine&&P.mine.p>0&&t){const k=Math.min(9,Math.floor(P.mine.p*10)),tl=TN['crack'+k],o=-.004;
  for(let f2=0;f2<6;f2++){for(let q=0;q<4;q++){const cc=FC[f2][q];DV.push(t.x+cc[0]+(cc[0]?-o:o),t.y+cc[1]+(cc[1]?-o:o),t.z+cc[2]+(cc[2]?-o:o),tu(tl,UVC[q][0]),tv(tl,UVC[q][1]),1,1,1,1,1)}DI.push(DN,DN+1,DN+2,DN,DN+2,DN+3);DN+=4}dflush()}
 gl.depthMask(true);gl.disable(gl.BLEND);
 drawSignText3D(cam);
}
 
/* ================= LOOP ================= */
let last=performance.now(),atkCd=0;
function update(dt,now){
 if(dim==0&&RULES.doDaylightCycle)tod=(tod+dt/600)%1;flash=Math.max(0,flash-dt);
 if(!spawned){if(pend)resolvePend();else placeSpawn()}
 const pcx=Math.floor(P.x/16),pcz=Math.floor(P.z/16);let g=0,me=0;
 for(const[dx,dz]of ORDER){const dd=dx*dx+dz*dz;if(dd>(RD+1)*(RD+1))break;const cx=pcx+dx,cz=pcz+dz,k=cx+','+cz;if(!chunks.has(k)&&g<3){const c=genChunk(cx,cz);chunks.set(k,c);initChunkLighting(c);g++}else{const c=chunks.get(k);if(c&&!c.sl)initChunkLighting(c)}}
 processLightingReset();processLighting(450);
 for(const[dx,dz]of ORDER){if(dx*dx+dz*dz>RD*RD)break;const c=chunks.get((pcx+dx)+','+(pcz+dz));if(c&&c.dirty&&!c.lightPending&&hasNb(c)){meshChunk(c);if(++me>=2)break}}
 if(Math.floor(now/1000)!=Math.floor((now-dt*1000)/1000))for(const[k,c]of chunks)if((c.cx-pcx)**2+(c.cz-pcz)**2>(RD+3)**2){freeMesh(c.mo);freeMesh(c.mt);const w=lightWorkChunks[dim],qi=w.indexOf(c);if(qi>=0)w.splice(qi,1);chunks.delete(k)}
 if(!spawned)return;
 if(!dead){
  updPlayer(dt);
  const ex=P.x,ey=P.y+1.62,ez=P.z,cp=Math.cos(P.pitch),dx=-Math.sin(P.yaw)*cp,dy=Math.sin(P.pitch),dz=-Math.cos(P.yaw)*cp,reach=gm==1?6:5;
  P.target=(gm==2||invOpen)?null:ray(ex,ey,ez,dx,dy,dz,reach);
  atkCd-=dt;P.placeCd=(P.placeCd||0)-dt;P.mineCd=(P.mineCd||0)-dt;
  if(locked&&!invOpen&&gm!=2){
   if(mouse.l){swing+=dt*14;const mh=rayMob(ex,ey,ez,dx,dy,dz,reach),t=P.target;
    if(mh&&(!t||mh.t<t.t)){P.mine=null;if(atkCd<=0){attack(mh.m);atkCd=.45}}
    else if(t){const b=BL[t.id];
     if(t.id==23)ignite(t.x,t.y,t.z);
     else if(b.hard<1e8){if(!P.mine||P.mine.x!=t.x||P.mine.y!=t.y||P.mine.z!=t.z)P.mine={x:t.x,y:t.y,z:t.z,p:0};
      const s=inv[sel],mi=mineInfo(b,s&&s.id);
      if(gm==1){if(P.mineCd<=0){breakBlock(t.x,t.y,t.z,false);P.mineCd=.2}}
      else{P.mine.p+=dt/mi.t*(P.fx.haste?1.5:1);if(P.mine.p>=1){breakBlock(t.x,t.y,t.z,mi.ok);P.mine=null}}}}
    else P.mine=null}
   else swing=0;
   if(mouse.r&&P.placeCd<=0){useItem();P.placeCd=.25}}
 }
 updEntities(dt);worldTick(dt);
 if(!initMobs&&spawned&&chunkAt(P.x,P.z)){initMobs=true;for(let i=0;i<14;i++){const a=Math.random()*6.28,r=6+Math.random()*22,x=P.x+Math.cos(a)*r,z=P.z+Math.sin(a)*r,y=groundY(x,z);if(y>0&&getBlock(Math.floor(x),y-1,Math.floor(z))==1)spawnMob(['pig','cow','sheep','chicken'][i%4],x,y,z)}}
}
const ORDER=[];for(let x=-12;x<=12;x++)for(let z=-12;z<=12;z++)ORDER.push([x,z]);ORDER.sort((a,b)=>a[0]*a[0]+a[1]*a[1]-b[0]*b[0]-b[1]*b[1]);
function frame(now){raf=requestAnimationFrame(frame);const dt=Math.min(.05,(now-last)/1000);last=now;fps+=(1/Math.max(dt,.001)-fps)*.05;
 update(dt,now);render(now);drawHUD(now)}
msg('Bem-vindo ao MineJS! Seed: '+SEED+'  (T = comandos · K = skin · O = texturas)');
raf=requestAnimationFrame(frame);
console.log('%cMineJS iniciado. Clique na tela para jogar. Pare com __mcStop()','color:#7d7;font-size:14px');console.log('Iluminação voxel incremental 0–15 ativa; modelos 3D de blocos restaurados no inventário.');
})();
 
