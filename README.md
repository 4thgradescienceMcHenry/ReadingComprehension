<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Banana Blast Royale</title>
<meta name="claude:viewport-fit-sides" content="cover">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Bungee&family=Nunito:wght@600;700;800;900&display=swap">
<style>
/* Layout: one-screen arcade. HUD bar on top, the island arena (canvas) in the middle,
   the answer bar at the bottom; reading, feedback and results open as overlay cards. */
:root{
  --sky:#0F1B3D;      /* night-sky navy (page ground) */
  --sky2:#1A2B5E;
  --peel:#FFD93B;     /* banana yellow (accent) */
  --peel-dk:#E0A800;
  --storm:#8E5CFF;    /* storm purple */
  --ink:#0B1230;
  --paper:#FFF8E1;    /* reading card */
  --text:#F4F6FF;
  --muted:#B9C3E6;
  --best:#2BD67B; --better:#3FA9F5; --good:#FFB23F; --miss:#FF6B8B;
  --display:"Bungee","Impact","Arial Black",sans-serif;
  --body:"Nunito","Trebuchet MS",Verdana,sans-serif;
  color-scheme:dark;
}
*{box-sizing:border-box}
[hidden]{display:none!important}
html,body{height:100%}
body{margin:0;background:radial-gradient(ellipse at 50% 0%,var(--sky2),var(--sky) 70%);color:var(--text);font-family:var(--body);overflow:hidden}
button{font-family:var(--body);font-weight:900;cursor:pointer;border:0}
button:focus-visible{outline:3px solid var(--peel);outline-offset:3px}
#app{height:100%;display:flex;flex-direction:column;padding-left:max(10px,env(safe-area-inset-left,0px));padding-right:max(10px,env(safe-area-inset-right,0px));padding-block:8px;gap:8px}
/* HUD */
#hud{display:flex;align-items:center;gap:10px;flex-wrap:wrap}
.brand{font-family:var(--display);font-size:clamp(14px,2.2vw,22px);color:var(--peel);letter-spacing:.5px;text-shadow:0 2px 0 #7a5200;margin-right:auto;white-space:nowrap}
.chip{background:rgba(255,255,255,.08);border:2px solid rgba(255,255,255,.14);border-radius:12px;padding:4px 10px;font-weight:900;font-size:14px;display:flex;align-items:center;gap:6px;white-space:nowrap;font-variant-numeric:tabular-nums}
.chip b{color:var(--peel)}
.shield{width:90px;height:10px;border-radius:6px;background:rgba(255,255,255,.15);overflow:hidden}
.shield i{display:block;height:100%;background:linear-gradient(90deg,#4FC3F7,#7C4DFF);width:100%;transition:width .2s}
.hbtn{background:var(--peel);color:var(--ink);border-radius:12px;padding:7px 12px;font-size:14px;box-shadow:0 3px 0 var(--peel-dk)}
.hbtn.ghost{background:rgba(255,255,255,.1);color:var(--text);box-shadow:none;border:2px solid rgba(255,255,255,.2)}
/* question + arena */
#qbar{background:rgba(11,18,48,.75);border:2px solid rgba(255,217,59,.45);border-radius:14px;padding:8px 14px;font-weight:800;font-size:clamp(15px,1.9vw,19px);line-height:1.3;min-height:44px;display:flex;align-items:center;gap:10px}
#qbar .tag{font-family:var(--display);font-size:12px;color:var(--ink);background:var(--peel);border-radius:8px;padding:3px 8px;white-space:nowrap}
#stage{flex:1;min-height:0;position:relative;display:flex;align-items:center;justify-content:center}
canvas{display:block;border-radius:18px;box-shadow:0 10px 40px rgba(0,0,0,.45);touch-action:none;max-width:100%;max-height:100%}
#choices{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:8px}
.choice{display:flex;align-items:flex-start;gap:8px;text-align:left;background:rgba(255,255,255,.07);border:2px solid rgba(255,255,255,.14);color:var(--text);border-radius:12px;padding:7px 10px;font-weight:700;font-size:clamp(13px,1.6vw,16px);line-height:1.3;min-width:0}
.choice:hover{border-color:var(--peel)}
.choice .lt{flex:none;width:26px;height:26px;border-radius:50%;display:grid;place-items:center;font-family:var(--display);font-size:14px;color:var(--ink)}
.choice span:last-child{min-width:0}
/* overlays */
.overlay{position:fixed;inset:0;background:rgba(6,10,30,.72);display:flex;align-items:center;justify-content:center;padding:16px max(16px,env(safe-area-inset-left,0px));z-index:20}
.card{background:var(--paper);color:var(--ink);border-radius:22px;max-width:720px;width:100%;max-height:100%;overflow:auto;padding:22px 24px;box-shadow:0 12px 0 rgba(0,0,0,.25);border:4px solid var(--ink)}
.card h1,.card h2{font-family:var(--display);margin:0 0 6px;line-height:1.05;text-wrap:balance}
.card h1{font-size:clamp(30px,6vw,52px);color:var(--ink)}
.card h1 span{color:#E59A00}
.card h2{font-size:clamp(20px,3.4vw,28px)}
.card p{font-size:17px;line-height:1.55;margin:8px 0}
.passage{background:#fff;border:2px dashed #E5C873;border-radius:14px;padding:12px 16px;font-size:18px;line-height:1.65;max-width:65ch}
.passage p{margin:0 0 10px;font-size:18px}
.eyebrow{font-weight:900;text-transform:uppercase;letter-spacing:1.5px;font-size:12px;color:#8A5A00}
.row{display:flex;gap:10px;flex-wrap:wrap;align-items:center;margin-top:12px}
.big{background:var(--peel);color:var(--ink);font-family:var(--display);font-size:20px;padding:12px 22px;border-radius:14px;box-shadow:0 5px 0 var(--peel-dk);border:3px solid var(--ink)}
.big:active{transform:translateY(3px);box-shadow:0 2px 0 var(--peel-dk)}
.sec{background:#fff;color:var(--ink);border:3px solid var(--ink);border-radius:14px;padding:10px 16px;font-size:16px}
.tiers{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:8px;margin:10px 0}
.tier{border-radius:12px;padding:8px;text-align:center;font-weight:900;color:var(--ink);font-size:14px;line-height:1.2}
.tier b{display:block;font-family:var(--display);font-size:18px}
.verdict{font-family:var(--display);font-size:clamp(28px,6vw,46px);margin:0}
.explain{background:#fff;border-radius:12px;padding:10px 14px;border:2px solid #E5C873}
.keys{font-size:14px;color:#5a4a20}
@media (max-width:560px){#choices{grid-template-columns:minmax(0,1fr)}.tiers{grid-template-columns:repeat(2,minmax(0,1fr))}.shield{width:60px}.brand{display:none}}
@media (prefers-reduced-motion:reduce){*{transition:none!important}}
</style>
</head>
<body>
<div id="app">
  <div id="hud">
    <div class="brand">BANANA BLAST ROYALE</div>
    <div class="chip">Score <b id="score">0</b></div>
    <div class="chip">Q <b id="qnum">0/18</b></div>
    <div class="chip">Shield <div class="shield"><i id="shieldbar"></i></div></div>
    <button class="hbtn ghost" id="readBtn" type="button">Re-read</button>
    <button class="hbtn ghost" id="soundBtn" type="button" aria-pressed="true">Sound on</button>
  </div>
  <div id="qbar"><span class="tag" id="skill">READY</span><span id="qtext">Drop onto the island to start your first reading mission!</span></div>
  <div id="stage"><canvas id="cv" width="960" height="560" aria-label="Game arena: move the banana and blast the answer balloon you think is best"></canvas></div>
  <div id="choices"></div>
</div>

<div class="overlay" id="ov"><div class="card" id="card"></div></div>

<script>
(()=>{
// ---------------------------------------------------------------- content
const PASSAGES=[
{title:"The Coral Gardeners",genre:"Informational",text:[
"Coral reefs look like rocks, but they are built by tiny living animals called polyps. Reefs give shelter to about one fourth of all ocean fish. Sadly, many reefs are being damaged by warmer water and pollution.",
"Some scientists have become “coral gardeners.” They snip small pieces of healthy coral and hang them on underwater frames that look like trees. In these nurseries, the pieces grow much faster than they would on the sea floor.",
"After several months, divers carefully attach the young corals to damaged reefs. It is slow work, but it is paying off. In some places, replanted reefs are once again full of colorful fish. The gardeners hope that, with help, reefs can recover for future generations."],
qs:[
{skill:"Main Idea",q:"What is the MAIN idea of the passage?",a:[
["Scientists grow coral in nurseries to help damaged reefs recover.","best","It covers the whole passage: the problem (damaged reefs) and the solution (growing and replanting coral)."],
["Coral gardeners plant young coral on reefs.","better","True, but it leaves out WHY they do it: to help damaged reefs recover."],
["Coral reefs are home to many ocean fish.","good","A true detail from paragraph 1, but it is not what the whole passage is mostly about."],
["Coral reefs are made of rocks and sand.","miss","The passage says reefs only LOOK like rocks. They are built by living animals."]]},
{skill:"Vocabulary",q:"In paragraph 2, the word “nurseries” most likely means…",a:[
["places where young corals are cared for until they are strong","best","Context clues: small pieces are hung on frames and “grow much faster.” A nursery is where young things are cared for as they grow."],
["places where coral is kept underwater","better","Close, but it misses the key idea of helping young coral GROW."],
["metal frames that look like trees","good","The frames are part of the nursery, but the word means the whole growing place."],
["shops that sell coral to divers","miss","Nothing in the passage talks about selling coral."]]},
{skill:"Inference",q:"Why do divers attach the young corals “carefully”?",a:[
["The young corals are fragile, and the reefs they are saving are already damaged.","best","This uses two clues: the corals are young and small, and the reefs are already hurt."],
["The young corals could break easily.","better","A smart inference! The best answer also connects it to the damaged reefs."],
["Divers have to move slowly underwater.","good","Possibly true, but the word “carefully” points to protecting the coral."],
["The scientists want to finish the job quickly.","miss","The passage says it is SLOW work, not fast."]]}]},
{title:"Maya’s Lemonade Stand",genre:"Fiction",text:[
"Maya was sure her lemonade stand would be the best on Maple Street. She used a fancy sign, glittery cups, and her secret recipe with extra sugar. By noon, only two people had bought a cup, and one of them was her grandpa.",
"Across the street, her neighbor Theo was selling plain lemonade in paper cups, and his line stretched down the sidewalk. Maya marched over, ready to complain. “What’s your secret?” she demanded.",
"Theo shrugged. “I asked people what they wanted. Most said it was too hot for sweet drinks, so I made mine extra cold and not too sugary.”",
"Maya walked home slowly. The next morning, she made a new sign: “Ice-Cold Lemonade — You Pick the Sweetness!”"],
qs:[
{skill:"Character",q:"How does Maya change from the beginning to the end?",a:[
["She goes from thinking she knows best to listening to what customers want.","best","It describes her change in thinking, not just her actions. Her new sign lets customers choose!"],
["She learns to make her lemonade less sweet.","better","True, but her bigger change is learning to listen to customers."],
["She makes a new sign for her stand.","good","This is something she DOES, but it doesn’t explain how SHE changes."],
["She decides to quit selling lemonade.","miss","She doesn’t quit. She makes a new sign the next morning!"]]},
{skill:"Theme",q:"Which sentence BEST states the theme of the story?",a:[
["Listening to others can help you succeed.","best","A theme is a life lesson. Theo succeeded by listening, and Maya learns to do the same."],
["Customers like cold drinks on hot days.","better","A true idea from the story, but it is about lemonade, not a life lesson."],
["Neighbors can be helpful.","good","Theo does help, but the story’s lesson is bigger than that."],
["Fancy things always sell better.","miss","The story shows the opposite. The fancy stand sold the least!"]]},
{skill:"Text Evidence",q:"Why did Theo have such a long line?",a:[
["He asked people what they wanted and made lemonade to match.","best","Theo says it himself: he asked people, then made it cold and not too sugary."],
["His lemonade was extra cold.","better","Part of the reason. The best answer includes how he KNEW to make it cold."],
["His stand was on the same street.","good","True, but Maya’s stand was on the same street too and she had no line."],
["He gave his lemonade away for free.","miss","The story never says that. He was selling it."]]}]},
{title:"Why Do Cats Purr?",genre:"Informational",text:[
"Most people think cats purr only when they are happy, but the truth is more surprising. Cats purr by quickly tightening and relaxing muscles in their voice box, about 25 to 150 times every second.",
"Happy cats curled in a lap do purr, but so do cats that are hurt, scared, or about to give birth. Scientists think purring may calm a cat the way humming calms some people. Some researchers have even found that the low sounds of purring might help bones and muscles heal faster.",
"Mother cats purr to guide their newborn kittens, which are born unable to see or hear well. Kittens can feel the vibrations and follow them to their mother. A purr, it turns out, can mean many different things."],
qs:[
{skill:"Author’s Purpose",q:"What is the author’s MAIN purpose?",a:[
["To explain the different reasons cats purr.","best","The whole passage gives reasons: calming, healing, and guiding kittens."],
["To give readers facts about cats.","better","True, but too general. The facts are all about PURRING."],
["To show that cats purr when they are happy.","good","That is the idea the author says is only part of the truth."],
["To convince readers to adopt a cat.","miss","The author is explaining, not persuading."]]},
{skill:"Key Details",q:"How do newborn kittens find their mother?",a:[
["They feel the vibrations of her purr and follow them.","best","Paragraph 3 says kittens can’t see or hear well, so they FEEL the purr and follow it."],
["Their mother purrs to guide them.","better","Correct! The best answer also explains HOW the kittens use the purr."],
["They stay close to warm things.","good","Sounds reasonable, but the passage doesn’t say it."],
["They listen for her loud meow.","miss","Newborn kittens can’t hear well, so this doesn’t fit."]]},
{skill:"Summary",q:"Which sentence BEST summarizes the passage?",a:[
["Cats purr for many reasons, like staying calm, healing, and guiding kittens.","best","A strong summary covers all the main points in a few words."],
["Cats purr when they are happy and when they are hurt.","better","Good, but it leaves out healing and guiding kittens."],
["Cats purr using muscles in their voice box.","good","An important detail, but only one part of the passage."],
["Cats only purr when they are curled in a lap.","miss","The passage says cats purr in many situations."]]}]},
{title:"Bikes Before Wings",genre:"Informational",text:[
"Before they ever flew, Wilbur and Orville Wright fixed bicycles. In 1892, the brothers opened a bicycle shop in Dayton, Ohio. They repaired bikes, and then they began building their own.",
"The work taught them important lessons. They learned how to make machines that were light but strong. They also noticed that a rider stays balanced by making tiny adjustments, leaning and steering all the time. The brothers believed a flying machine would need the same kind of control.",
"Money from the shop paid for their experiments with gliders. In 1903, near Kitty Hawk, North Carolina, Orville flew their airplane for 12 seconds. It was the first powered, controlled airplane flight. The skills from a small bike shop had helped change the world."],
qs:[
{skill:"Cause & Effect",q:"How did the bicycle shop help the Wright brothers fly?",a:[
["It taught them about balance and strong, light machines, and it paid for their experiments.","best","This includes ALL the ways the shop helped: lessons AND money."],
["It gave them money for their glider experiments.","better","True, but the shop also taught them important lessons."],
["It was located in Dayton, Ohio.","good","A true detail, but it doesn’t explain how the shop helped them fly."],
["It sold airplanes to customers.","miss","They sold and fixed bikes, not airplanes."]]},
{skill:"Text Evidence",q:"Which detail BEST supports the idea that riding a bike is like flying a plane?",a:[
["A rider stays balanced by making tiny adjustments, leaning and steering.","best","The brothers believed a plane needed this SAME kind of control."],
["They learned to make machines that were light but strong.","better","A useful lesson for planes, but it is about building, not riding or flying."],
["The brothers repaired bikes and built their own.","good","True, but it doesn’t compare riding to flying."],
["The first flight lasted 12 seconds.","miss","A fact about the flight, not about how bikes and planes are alike."]]},
{skill:"Vocabulary",q:"In paragraph 2, the word “adjustments” means…",a:[
["small changes made to keep something working just right","best","Riders make tiny changes, leaning and steering, to stay balanced."],
["changes","better","Correct but incomplete. The context shows they are SMALL changes for balance."],
["leaning and steering","good","These are examples of adjustments, not what the word means."],
["repairs that cost money","miss","The word isn’t about paying for repairs here."]]}]},
{title:"The Night the Lights Went Out",genre:"Fiction",text:[
"When the storm knocked out the power, Jayden groaned. No video games, no TV, and his tablet battery was at three percent. “This is the worst night ever,” he muttered.",
"Mom lit candles and set them on the kitchen table. Dad dug an old board game out of the closet, its box held together with tape. At first, Jayden played with his arms crossed.",
"Then his little sister, Kiara, landed on the space that sent her all the way back to start, and she laughed so hard that milk came out of her nose. Jayden laughed too. They played three more games and told stories by candlelight until long past bedtime.",
"When the lights flickered back on, Jayden felt something strange. He was almost disappointed."],
qs:[
{skill:"Inference",q:"Why is Jayden “almost disappointed” when the lights come back on?",a:[
["He enjoyed the time with his family and didn’t want it to end.","best","Clues: laughing, three more games, stories past bedtime. The family time became special."],
["He was having fun playing the board game.","better","True, but it was more than the game. It was the time together."],
["He was tired from staying up so late.","good","He did stay up late, but that would not make him disappointed."],
["He wanted the storm to get even worse.","miss","Nothing in the story suggests he wanted that."]]},
{skill:"Character",q:"Which word BEST describes Jayden at the beginning of the story?",a:[
["grumpy","best","He groans, mutters “worst night ever,” and crosses his arms. That’s grumpy!"],
["bored","better","He might be bored, but his groaning and crossed arms show he is grumpy."],
["tired","good","The story doesn’t give clues that he is tired at the start."],
["excited","miss","He calls it the worst night ever, the opposite of excited."]]},
{skill:"Theme",q:"What is the theme of the story?",a:[
["Unplanned time with family can turn into the best memories.","best","Jayden expected the worst night, but it turned into a special one."],
["Board games can be really fun.","better","True in the story, but the lesson is about family and surprises."],
["Storms can knock out the power.","good","This is what happens in the plot, not the life lesson."],
["Technology is always better than games.","miss","The story shows Jayden having fun WITHOUT technology."]]}]},
{title:"Keep Our Library Open on Sundays",genre:"Persuasive Letter",text:[
"Dear Editor,",
"The city plans to close the Oakwood Library on Sundays to save money. I believe this is a mistake. Sunday is the only day many working families can visit together. Last year, over 300 people attended Sunday story time and homework help. For students without internet at home, the library is the best place to finish assignments.",
"Closing on Sundays would save the city about $40,000 a year, but that is less than one percent of the city’s budget. Libraries help children become stronger readers, and strong readers grow into successful adults.",
"Please ask the city council to find savings somewhere else. Our library is worth it.",
"Sincerely, Ana Ruiz, 5th grader"],
qs:[
{skill:"Author’s Purpose",q:"What is Ana’s MAIN purpose for writing this letter?",a:[
["To persuade readers to keep the library open on Sundays.","best","She states her opinion, gives reasons, and asks readers to act."],
["To explain why the library matters to families.","better","She does explain this, but she explains it in order to PERSUADE."],
["To inform readers about the city’s budget.","good","She mentions the budget only to support her argument."],
["To entertain readers with a story about the library.","miss","This is a letter with an argument, not a story."]]},
{skill:"Fact & Opinion",q:"Which is a FACT Ana uses to support her argument?",a:[
["Over 300 people attended Sunday story time and homework help last year.","best","It can be checked AND it strongly supports keeping Sundays open."],
["Closing on Sundays would save about $40,000 a year.","better","This is a fact, but it supports the city’s side. She has to answer it."],
["Sunday is the only day many working families can visit together.","good","This is a claim that is hard to prove for everyone."],
["Our library is worth it.","miss","This is Ana’s opinion, not a fact."]]},
{skill:"Argument",q:"How does Ana respond to the idea that closing will save money?",a:[
["She points out the savings are less than one percent of the budget.","best","She uses a number to show the savings are very small."],
["She asks the council to find savings somewhere else.","better","She does, but first she shows WHY the savings aren’t worth it."],
["She says libraries help children read.","good","That is another reason, but it doesn’t answer the money argument."],
["She agrees the library should close.","miss","She says closing is a mistake."]]}]}
];
const TIER={best:{label:"BEST ANSWER!",pts:100,color:"#2BD67B"},better:{label:"BETTER ANSWER",pts:60,color:"#3FA9F5"},good:{label:"GOOD TRY",pts:25,color:"#FFB23F"},miss:{label:"NOT QUITE",pts:0,color:"#FF6B8B"}};
const LETTERS=["A","B","C","D"], LCOL=["#FF5FA2","#3FA9F5","#2BD67B","#FFB23F"];

// ---------------------------------------------------------------- state
const cv=document.getElementById('cv'), ctx=cv.getContext('2d');
const W=960,H=560;
const $=id=>document.getElementById(id);
let S={p:0,q:0,score:0,shield:100,bests:0,betters:0,answered:0,phase:'title'};
let player={x:W/2,y:H/2+60,vx:0,vy:0,face:1,aim:0,cool:0,hurt:0,drop:0};
let shots=[],targets=[],bots=[],fx=[],choiceOrder=[],storm=1,time=0,paused=true;
const keys={};let mouse={x:W/2,y:H/2,down:false};
let joy=null, sound=true, actx=null;
const rnd=(a,b)=>a+Math.random()*(b-a);

// ---------------------------------------------------------------- audio
function beep(f,d,type='square',v=.06,slide=0){if(!sound)return;try{actx=actx||new (window.AudioContext||window.webkitAudioContext)();const o=actx.createOscillator(),g=actx.createGain();o.type=type;o.frequency.value=f;if(slide)o.frequency.exponentialRampToValueAtTime(Math.max(40,f+slide),actx.currentTime+d);g.gain.value=v;g.gain.exponentialRampToValueAtTime(.0001,actx.currentTime+d);o.connect(g).connect(actx.destination);o.start();o.stop(actx.currentTime+d)}catch(e){}}
const sfx={pew:()=>beep(880,.08,'square',.035,-500),pop:()=>{beep(520,.12,'triangle',.08,500);setTimeout(()=>beep(990,.1,'triangle',.06),60)},bot:()=>beep(300,.15,'sawtooth',.04,-200),hurt:()=>beep(160,.2,'square',.05,-60),win:()=>[523,659,784,1046].forEach((f,i)=>setTimeout(()=>beep(f,.18,'triangle',.07),i*110)),miss:()=>beep(220,.3,'sine',.07,-120)};

// ---------------------------------------------------------------- UI
function hud(){$('score').textContent=S.score;$('qnum').textContent=`${S.answered}/18`;$('shieldbar').style.width=Math.max(0,S.shield)+'%'}
function show(html){$('card').innerHTML=html;$('ov').hidden=false;paused=true;const b=$('card').querySelector('.big');if(b)b.focus()}
function hide(){$('ov').hidden=true;paused=false}
function curP(){return PASSAGES[S.p]}
function curQ(){return curP().qs[S.q]}

function titleScreen(){
  S.phase='title';
  show(`<p class="eyebrow">Reading mission · 5th grade</p>
  <h1>BANANA <span>BLAST</span> ROYALE</h1>
  <p>Agent Banana is dropping onto Comprehension Island! Read each passage, then <b>blast the answer balloon</b> you think is the <b>BEST</b> answer. Some answers are good, some are better, and only one is the best.</p>
  <div class="tiers">
    <div class="tier" style="background:var(--best)"><b>BEST</b>+100 &amp; shield</div>
    <div class="tier" style="background:var(--better)"><b>BETTER</b>+60</div>
    <div class="tier" style="background:var(--good)"><b>GOOD</b>+25</div>
    <div class="tier" style="background:var(--miss)"><b>MISS</b>+0</div>
  </div>
  <p class="keys"><b>Computer:</b> move with WASD or the arrow keys · aim with the mouse · click to blast.<br><b>Tablet or phone:</b> drag on the left side to move · tap anywhere else to blast.<br>Blast the purple Glitch Bots for +10 bonus. They pop into confetti!</p>
  <div class="row"><button class="big" id="go" type="button">DROP IN!</button></div>`);
  $('go').onclick=()=>{sfx.win();readScreen()};
}
function passageHTML(){const P=curP();return `<div class="passage">${P.text.map(t=>`<p>${t}</p>`).join('')}</div>`}
function readScreen(again){
  S.phase='read';
  const P=curP();
  show(`<p class="eyebrow">Passage ${S.p+1} of ${PASSAGES.length} · ${P.genre}</p><h2>${P.title}</h2>${passageHTML()}
  <div class="row"><button class="big" id="go" type="button">${again?'BACK TO BATTLE':'READY! START BATTLE'}</button></div>`);
  $('go').onclick=()=>{hide();if(!again)startQuestion()};
}
function startQuestion(){
  S.phase='battle';
  const Q=curQ();
  $('skill').textContent=Q.skill.toUpperCase();$('qtext').textContent=Q.q;
  choiceOrder=[0,1,2,3].sort(()=>Math.random()-.5);
  $('choices').innerHTML=choiceOrder.map((ai,i)=>`<button class="choice" type="button" data-i="${i}"><span class="lt" style="background:${LCOL[i]}">${LETTERS[i]}</span><span>${Q.a[ai][0]}</span></button>`).join('');
  $('choices').querySelectorAll('.choice').forEach(b=>b.onclick=()=>autoFire(+b.dataset.i));
  // targets
  targets=choiceOrder.map((ai,i)=>({i,ai,x:160+i*215,y:rnd(110,200),vx:rnd(-40,40),vy:rnd(-30,30),r:38,ph:rnd(0,6),alive:true}));
  const nb=Math.min(2+S.answered,6);bots=[];
  for(let k=0;k<nb;k++)spawnBot();
  storm=1;player.drop=S.answered===0||S.q===0?1:0;if(player.drop){player.x=W/2;player.y=H/2+60}
  hud();paused=false;
}
function spawnBot(){const side=Math.floor(rnd(0,4));bots.push({x:side<2?rnd(60,W-60):(side==2?40:W-40),y:side<2?(side?H-40:40):rnd(60,H-60),r:16,hp:2,t:rnd(0,6),sp:rnd(38,60)})}
function answer(t){
  const Q=curQ();const [txt,tier,why]=Q.a[t.ai];const T=TIER[tier];
  S.score+=T.pts;S.answered++;
  if(tier==='best'){S.bests++;S.shield=Math.min(100,S.shield+25);sfx.win()}else if(tier==='miss'){sfx.miss()}else sfx.pop();
  if(tier==='better')S.betters++;
  const bestA=Q.a.find(a=>a[1]==='best');const bestLetter=LETTERS[choiceOrder.findIndex(ai=>Q.a[ai][1]==='best')];
  hud();
  setTimeout(()=>{
    show(`<p class="verdict" style="color:${T.color};-webkit-text-stroke:2px var(--ink)">${T.label}</p>
    <p><b>+${T.pts} points.</b> You chose: “${txt}”</p>
    <div class="explain"><p><b>Why:</b> ${why}</p>${tier!=='best'?`<p><b>The BEST answer was ${bestLetter}:</b> “${bestA[0]}”<br>${bestA[2]}</p>`:''}</div>
    <div class="row"><button class="big" id="go" type="button">${S.answered>=18?'SEE RESULTS':'NEXT QUESTION'}</button></div>`);
    $('go').onclick=next;
  },550);
}
function next(){
  S.q++;
  if(S.q>=curP().qs.length){S.q=0;S.p++;if(S.p>=PASSAGES.length)return endScreen();hide();return readScreen()}
  hide();startQuestion();
}
function endScreen(){
  S.phase='end';
  const max=1800,pct=Math.round(S.score/max*100);
  const rank=pct>=90?'GOLDEN BANANA LEGEND':pct>=75?'TOP BANANA':pct>=55?'BANANA SQUAD CAPTAIN':'BANANA ROOKIE';
  show(`<p class="eyebrow">Mission complete</p><h1>BANANA <span>ROYALE</span> CHAMPION!</h1>
  <p style="font-size:22px"><b>${S.score}</b> / ${max} points</p>
  <div class="tiers" style="grid-template-columns:repeat(2,minmax(0,1fr))"><div class="tier" style="background:var(--best)"><b>${S.bests}</b>BEST answers</div><div class="tier" style="background:var(--better)"><b>${S.betters}</b>BETTER answers</div></div>
  <p>Your rank: <b>${rank}</b></p>
  <p>Remember the secret: a BEST answer is complete, uses evidence from the text, and answers exactly what the question asks.</p>
  <div class="row"><button class="big" id="go" type="button">PLAY AGAIN</button></div>`);
  $('go').onclick=()=>{S={p:0,q:0,score:0,shield:100,bests:0,betters:0,answered:0,phase:'title'};hud();readScreen()};
  $('choices').innerHTML='';$('qtext').textContent='Mission complete! Great reading, Agent Banana.';$('skill').textContent='DONE';
}
function autoFire(i){const t=targets.find(t=>t.i===i&&t.alive);if(!t||paused)return;fire(Math.atan2(t.y-player.y,t.x-player.x),true)}

// ---------------------------------------------------------------- input
addEventListener('keydown',e=>{keys[e.key.toLowerCase()]=true;if(['arrowup','arrowdown','arrowleft','arrowright',' '].includes(e.key.toLowerCase())&&S.phase==='battle'&&!paused)e.preventDefault()});
addEventListener('keyup',e=>keys[e.key.toLowerCase()]=false);
function toWorld(cx,cy){const r=cv.getBoundingClientRect();return{x:(cx-r.left)/r.width*W,y:(cy-r.top)/r.height*H}}
cv.addEventListener('mousemove',e=>{const p=toWorld(e.clientX,e.clientY);mouse.x=p.x;mouse.y=p.y});
cv.addEventListener('mousedown',e=>{mouse.down=true;const p=toWorld(e.clientX,e.clientY);mouse.x=p.x;mouse.y=p.y});
addEventListener('mouseup',()=>mouse.down=false);
cv.addEventListener('touchstart',e=>{e.preventDefault();for(const t of e.changedTouches){const p=toWorld(t.clientX,t.clientY);if(p.x<W*.35&&!joy){joy={id:t.identifier,ox:p.x,oy:p.y,x:p.x,y:p.y}}else{mouse.x=p.x;mouse.y=p.y;fire(Math.atan2(p.y-player.y,p.x-player.x))}}},{passive:false});
cv.addEventListener('touchmove',e=>{e.preventDefault();for(const t of e.changedTouches){if(joy&&t.identifier===joy.id){const p=toWorld(t.clientX,t.clientY);joy.x=p.x;joy.y=p.y}}},{passive:false});
cv.addEventListener('touchend',e=>{for(const t of e.changedTouches)if(joy&&t.identifier===joy.id)joy=null});
$('readBtn').onclick=()=>{if(S.phase==='battle'||S.phase==='read')readScreen(S.phase==='battle')};
$('soundBtn').onclick=()=>{sound=!sound;$('soundBtn').textContent=sound?'Sound on':'Sound off';$('soundBtn').setAttribute('aria-pressed',sound)};

function fire(ang,force){
  if(paused||S.phase!=='battle'||player.drop>0)return;
  if(player.cool>0&&!force)return;
  player.cool=.22;player.aim=ang;player.face=Math.cos(ang)>=0?1:-1;
  const gx=player.x+Math.cos(ang)*34,gy=player.y-6+Math.sin(ang)*34;
  shots.push({x:gx,y:gy,vx:Math.cos(ang)*620,vy:Math.sin(ang)*620,life:1.2,c:['#FFD93B','#FF5FA2','#7CF29A','#4FC3F7'][Math.floor(rnd(0,4))]});
  sfx.pew();
}
function confetti(x,y,n,cols){for(let k=0;k<n;k++){const a=rnd(0,6.28),s=rnd(80,320);fx.push({x,y,vx:Math.cos(a)*s,vy:Math.sin(a)*s-60,life:rnd(.6,1.2),c:cols[k%cols.length],r:rnd(3,6),rot:rnd(0,6)})}}

// ---------------------------------------------------------------- update
function update(dt){
  time+=dt;
  if(paused)return;
  if(player.drop>0){player.drop-=dt*.8;return}
  let ax=0,ay=0;
  if(keys['a']||keys['arrowleft'])ax--;if(keys['d']||keys['arrowright'])ax++;
  if(keys['w']||keys['arrowup'])ay--;if(keys['s']||keys['arrowdown'])ay++;
  if(joy){const dx=joy.x-joy.ox,dy=joy.y-joy.oy,m=Math.hypot(dx,dy);if(m>6){ax=dx/Math.max(m,40);ay=dy/Math.max(m,40)}}
  const m=Math.hypot(ax,ay);if(m>1){ax/=m;ay/=m}
  player.x=Math.max(40,Math.min(W-40,player.x+ax*260*dt));player.y=Math.max(70,Math.min(H-30,player.y+ay*260*dt));
  if(Math.abs(ax)>.1&&!mouse.down)player.face=ax>0?1:-1;
  player.cool=Math.max(0,player.cool-dt);player.hurt=Math.max(0,player.hurt-dt);
  const aimA=Math.atan2(mouse.y-player.y,mouse.x-player.x);player.aim=aimA;if(mouse.down){player.face=Math.cos(aimA)>=0?1:-1;fire(aimA)}
  if(keys[' ']){const t=targets.find(t=>t.alive);if(t)fire(player.aim)}
  storm=Math.max(.62,storm-dt*.004);
  for(const t of targets){if(!t.alive)continue;t.ph+=dt*2;t.x+=t.vx*dt;t.y+=t.vy*dt;if(t.x<60||t.x>W-60)t.vx*=-1;if(t.y<80||t.y>H-140)t.vy*=-1;t.x=Math.max(60,Math.min(W-60,t.x));t.y=Math.max(80,Math.min(H-140,t.y))}
  for(const b of bots){b.t+=dt;const dx=player.x-b.x,dy=player.y-b.y,d=Math.hypot(dx,dy)||1;b.x+=dx/d*b.sp*dt+Math.sin(b.t*3)*20*dt;b.y+=dy/d*b.sp*dt;
    if(d<b.r+26&&player.hurt<=0){S.shield-=12;player.hurt=1;sfx.hurt();player.x+=dx/d*40;player.y+=dy/d*40;b.x-=dx/d*60;b.y-=dy/d*60;confetti(player.x,player.y,8,['#4FC3F7','#B388FF']);
      if(S.shield<=0){S.shield=100;S.score=Math.max(0,S.score-30);fx.push({text:'SHIELD RESET −30',x:player.x,y:player.y-60,life:1.6,c:'#FF6B8B'})}hud()}}
  for(const s of shots){s.x+=s.vx*dt;s.y+=s.vy*dt;s.life-=dt;
    for(const t of targets){if(t.alive&&Math.hypot(s.x-t.x,s.y-t.y)<t.r+6){t.alive=false;s.life=0;confetti(t.x,t.y,40,[LCOL[t.i],'#FFFFFF','#FFD93B']);answer(t);paused=true;break}}
    for(const b of bots){if(b.hp>0&&s.life>0&&Math.hypot(s.x-b.x,s.y-b.y)<b.r+6){s.life=0;b.hp--;if(b.hp<=0){S.score+=10;hud();sfx.bot();confetti(b.x,b.y,22,['#B388FF','#7CF29A','#FFFFFF']);fx.push({text:'+10',x:b.x,y:b.y-20,life:1,c:'#7CF29A'})}}}}
  shots=shots.filter(s=>s.life>0&&s.x>-20&&s.x<W+20&&s.y>-20&&s.y<H+20);
  bots=bots.filter(b=>b.hp>0);
  if(bots.length<2&&Math.random()<dt*.5)spawnBot();
}
function updateFx(dt){for(const f of fx){f.life-=dt;if(f.text){f.y-=30*dt;continue}f.x+=f.vx*dt;f.y+=f.vy*dt;f.vy+=300*dt;f.vx*=.98;f.rot+=dt*8}fx=fx.filter(f=>f.life>0)}

// ---------------------------------------------------------------- draw
const trees=[[90,120],[860,110],[120,460],[840,470],[480,90],[300,500],[680,520],[60,300],[910,300]].map(([x,y],i)=>({x,y,r:26+(i%3)*6}));
const rocks=[[230,330],[740,250],[600,380],[380,240]];
function drawIsland(){
  // water
  const g=ctx.createLinearGradient(0,0,0,H);g.addColorStop(0,'#2E8BC0');g.addColorStop(1,'#145DA0');ctx.fillStyle=g;ctx.fillRect(0,0,W,H);
  ctx.strokeStyle='rgba(255,255,255,.25)';ctx.lineWidth=3;
  for(let k=0;k<14;k++){const y=(k*47+time*12)%H,x=(k*137)%W;ctx.beginPath();ctx.arc(x,y,14,Math.PI*1.1,Math.PI*1.9);ctx.stroke()}
  // sand + grass
  ctx.fillStyle='#F6D98B';roundBlob(W/2,H/2+6,W*.47,H*.47);ctx.fillStyle='#5CC46B';roundBlob(W/2,H/2+6,W*.44,H*.42);
  ctx.fillStyle='#4DB35D';for(let k=0;k<40;k++){const x=(k*97)%W,y=60+(k*53)%(H-110);ctx.fillRect(x,y,6,3)}
  // path
  ctx.strokeStyle='rgba(214,180,110,.55)';ctx.lineWidth=26;ctx.lineCap='round';ctx.beginPath();ctx.moveTo(140,420);ctx.quadraticCurveTo(480,250,820,400);ctx.stroke();
  for(const[x,y]of rocks){ctx.fillStyle='#8C96A8';ctx.beginPath();ctx.ellipse(x,y,26,18,0,0,7);ctx.fill();ctx.fillStyle='#AEB7C6';ctx.beginPath();ctx.ellipse(x-6,y-5,12,7,0,0,7);ctx.fill()}
  for(const t of trees){ctx.fillStyle='rgba(0,0,0,.18)';ctx.beginPath();ctx.ellipse(t.x+6,t.y+t.r*.8,t.r,t.r*.4,0,0,7);ctx.fill();ctx.fillStyle='#2E8B4E';ctx.beginPath();ctx.arc(t.x,t.y,t.r,0,7);ctx.fill();ctx.fillStyle='#3FAE63';ctx.beginPath();ctx.arc(t.x-t.r*.3,t.y-t.r*.3,t.r*.55,0,7);ctx.fill()}
}
function roundBlob(cx,cy,rx,ry){ctx.beginPath();for(let k=0;k<=40;k++){const a=k/40*6.283,w=1+.05*Math.sin(a*5)+.03*Math.sin(a*9);const x=cx+Math.cos(a)*rx*w,y=cy+Math.sin(a)*ry*w;k?ctx.lineTo(x,y):ctx.moveTo(x,y)}ctx.fill()}
function drawStorm(){
  ctx.save();ctx.beginPath();ctx.rect(0,0,W,H);ctx.ellipse(W/2,H/2,W*.7*storm,H*.7*storm,0,0,7,true);ctx.fillStyle='rgba(142,92,255,.28)';ctx.fill('evenodd');
  ctx.strokeStyle='rgba(200,170,255,.8)';ctx.lineWidth=4;ctx.setLineDash([14,10]);ctx.lineDashOffset=-time*30;ctx.beginPath();ctx.ellipse(W/2,H/2,W*.7*storm,H*.7*storm,0,0,7);ctx.stroke();ctx.restore();
}
function drawBanana(p){
  const drop=p.drop>0?p.drop:0;
  ctx.save();ctx.translate(p.x,p.y-drop*420);
  // parachute
  if(drop>0){ctx.strokeStyle='#fff';ctx.lineWidth=2;for(const sx of[-40,0,40]){ctx.beginPath();ctx.moveTo(sx,-110);ctx.lineTo(0,-40);ctx.stroke()}ctx.fillStyle='#FF5FA2';ctx.beginPath();ctx.ellipse(0,-112,62,30,0,Math.PI,0);ctx.fill();ctx.fillStyle='#FFD93B';ctx.beginPath();ctx.ellipse(0,-112,22,30,0,Math.PI,0);ctx.fill()}
  // shadow
  if(!drop){ctx.fillStyle='rgba(0,0,0,.22)';ctx.beginPath();ctx.ellipse(0,30,26,8,0,0,7);ctx.fill()}
  if(p.hurt>0&&Math.floor(time*20)%2)ctx.globalAlpha=.55;
  ctx.scale(p.face,1);
  // legs
  ctx.strokeStyle='#5A3A10';ctx.lineWidth=5;ctx.lineCap='round';const step=Math.sin(time*12)*4;
  ctx.beginPath();ctx.moveTo(-6,16);ctx.lineTo(-9,28+step);ctx.moveTo(8,16);ctx.lineTo(10,28-step);ctx.stroke();
  ctx.fillStyle='#FF5FA2';ctx.beginPath();ctx.ellipse(-11,30+step,7,4,0,0,7);ctx.ellipse(12,30-step,7,4,0,0,7);ctx.fill();
  // banana body: a curved crescent
  ctx.fillStyle='#FFD93B';ctx.strokeStyle='#8A5A00';ctx.lineWidth=3;
  ctx.beginPath();ctx.moveTo(-4,-44);ctx.bezierCurveTo(26,-30,30,10,4,24);ctx.bezierCurveTo(-8,30,-20,22,-14,14);ctx.bezierCurveTo(4,0,2,-26,-12,-40);ctx.closePath();ctx.fill();ctx.stroke();
  ctx.fillStyle='#FFE97A';ctx.beginPath();ctx.moveTo(2,-36);ctx.bezierCurveTo(18,-22,20,0,6,14);ctx.bezierCurveTo(10,0,10,-20,2,-36);ctx.fill();
  // tips
  ctx.fillStyle='#5A3A10';ctx.beginPath();ctx.ellipse(-8,-44,5,4,-.6,0,7);ctx.fill();ctx.beginPath();ctx.ellipse(-16,16,4,3,.4,0,7);ctx.fill();
  // backwards cap
  ctx.fillStyle='#3FA9F5';ctx.beginPath();ctx.ellipse(0,-36,13,8,-.3,Math.PI,0);ctx.fill();ctx.fillRect(-16,-38,10,4);
  // sunglasses
  ctx.fillStyle='#14141F';ctx.beginPath();ctx.roundRect(-2,-22,13,9,3);ctx.roundRect(13,-20,11,9,3);ctx.fill();ctx.fillRect(9,-19,6,2);
  ctx.fillStyle='rgba(255,255,255,.7)';ctx.fillRect(1,-20,4,2);ctx.fillRect(15,-18,3,2);
  // smile
  ctx.strokeStyle='#5A3A10';ctx.lineWidth=2.5;ctx.beginPath();ctx.arc(12,-6,6,.2,Math.PI-.6);ctx.stroke();
  ctx.restore();
  // blaster arm (not mirrored so it follows the aim)
  if(!drop){ctx.save();ctx.translate(p.x+p.face*4,p.y-4);ctx.rotate(p.aim);
    ctx.strokeStyle='#5A3A10';ctx.lineWidth=5;ctx.beginPath();ctx.moveTo(0,0);ctx.lineTo(14,0);ctx.stroke();
    ctx.fillStyle='#FF8A00';ctx.strokeStyle='#14141F';ctx.lineWidth=2.5;ctx.beginPath();ctx.roundRect(12,-7,26,13,4);ctx.fill();ctx.stroke();
    ctx.fillStyle='#4FC3F7';ctx.fillRect(34,-4,8,7);ctx.restore()}
}
function drawTarget(t){
  const bob=Math.sin(t.ph)*6;ctx.save();ctx.translate(t.x,t.y+bob);
  ctx.strokeStyle='rgba(255,255,255,.8)';ctx.lineWidth=2;ctx.beginPath();ctx.moveTo(0,t.r);ctx.quadraticCurveTo(8,t.r+20,0,t.r+40);ctx.stroke();
  ctx.fillStyle='#8B5A2B';ctx.fillRect(-14,t.r+38,28,18);ctx.strokeStyle='#5A3A10';ctx.strokeRect(-14,t.r+38,28,18);
  ctx.fillStyle=LCOL[t.i];ctx.beginPath();ctx.arc(0,0,t.r,0,7);ctx.fill();ctx.strokeStyle='#0B1230';ctx.lineWidth=3;ctx.stroke();
  ctx.fillStyle='rgba(255,255,255,.4)';ctx.beginPath();ctx.ellipse(-12,-14,9,14,-.5,0,7);ctx.fill();
  ctx.fillStyle='#0B1230';ctx.font='30px Bungee, Impact, sans-serif';ctx.textAlign='center';ctx.textBaseline='middle';ctx.fillText(LETTERS[t.i],0,3);
  ctx.restore();
}
function drawBot(b){ctx.save();ctx.translate(b.x,b.y+Math.sin(b.t*6)*2);
  ctx.fillStyle='rgba(0,0,0,.2)';ctx.beginPath();ctx.ellipse(0,20,14,5,0,0,7);ctx.fill();
  ctx.fillStyle=b.hp>1?'#8E5CFF':'#C3A6FF';ctx.strokeStyle='#2A1260';ctx.lineWidth=3;ctx.beginPath();ctx.roundRect(-16,-16,32,30,8);ctx.fill();ctx.stroke();
  ctx.fillStyle='#14141F';ctx.fillRect(-11,-8,22,10);ctx.fillStyle='#7CF29A';ctx.fillRect(-7+Math.sin(b.t*4)*4,-6,6,6);
  ctx.strokeStyle='#2A1260';ctx.beginPath();ctx.moveTo(0,-16);ctx.lineTo(0,-24);ctx.stroke();ctx.fillStyle='#FF5FA2';ctx.beginPath();ctx.arc(0,-26,4,0,7);ctx.fill();
  ctx.restore()}
function draw(){
  ctx.clearRect(0,0,W,H);drawIsland();
  for(const t of targets)if(t.alive)drawTarget(t);
  for(const b of bots)drawBot(b);
  for(const s of shots){ctx.fillStyle=s.c;ctx.beginPath();ctx.arc(s.x,s.y,6,0,7);ctx.fill();ctx.fillStyle='rgba(255,255,255,.7)';ctx.beginPath();ctx.arc(s.x-2,s.y-2,2,0,7);ctx.fill()}
  drawBanana(player);
  for(const f of fx){ctx.globalAlpha=Math.min(1,f.life*2);if(f.text){ctx.fillStyle=f.c;ctx.font='20px Bungee, Impact, sans-serif';ctx.textAlign='center';ctx.fillText(f.text,f.x,f.y)}else{ctx.save();ctx.translate(f.x,f.y);ctx.rotate(f.rot);ctx.fillStyle=f.c;ctx.fillRect(-f.r,-f.r/2,f.r*2,f.r);ctx.restore()}ctx.globalAlpha=1}
  drawStorm();
  // crosshair
  if(S.phase==='battle'&&!paused){ctx.strokeStyle='#FFFFFF';ctx.lineWidth=2;ctx.beginPath();ctx.arc(mouse.x,mouse.y,10,0,7);ctx.moveTo(mouse.x-16,mouse.y);ctx.lineTo(mouse.x-5,mouse.y);ctx.moveTo(mouse.x+5,mouse.y);ctx.lineTo(mouse.x+16,mouse.y);ctx.stroke()}
  if(joy){ctx.strokeStyle='rgba(255,255,255,.6)';ctx.lineWidth=3;ctx.beginPath();ctx.arc(joy.ox,joy.oy,40,0,7);ctx.stroke();ctx.fillStyle='rgba(255,217,59,.8)';ctx.beginPath();ctx.arc(joy.x,joy.y,18,0,7);ctx.fill()}
  if(player.drop>0){ctx.fillStyle='rgba(11,18,48,.6)';ctx.font='26px Bungee, Impact, sans-serif';ctx.textAlign='center';ctx.fillStyle='#FFD93B';ctx.fillText('DROPPING IN...',W/2,H-40)}
}
// ---------------------------------------------------------------- sizing + loop
function fit(){const st=$('stage');const r=st.getBoundingClientRect();const s=Math.min(r.width/W,r.height/H);cv.style.width=Math.floor(W*s)+'px';cv.style.height=Math.floor(H*s)+'px'}
addEventListener('resize',fit);new ResizeObserver(fit).observe($('stage'));
let last=performance.now();
function loop(now){const dt=Math.min(.05,(now-last)/1000);last=now;update(dt);updateFx(dt);draw();requestAnimationFrame(loop)}
fit();hud();titleScreen();requestAnimationFrame(loop);
})();
</script>
</body>
</html>
