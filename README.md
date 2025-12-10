<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>オセロニアポーカー — 単一ファイル完全版</title>
<style>
  :root{--bg:#f6f6f8;--card:#fff;--accent:#2b6df6}
  body{font-family:system-ui,-apple-system,"Hiragino Kaku Gothic ProN","メイリオ",sans-serif;margin:0;background:var(--bg);color:#111}
  header{background:#111;color:#fff;padding:10px 16px}
  .wrap{max-width:1100px;margin:18px auto;padding:12px;background:#fff;border-radius:8px;box-shadow:0 4px 18px rgba(0,0,0,.06)}
  h1{margin:0 0 12px 0;font-size:1.2rem}
  .row{display:flex;gap:12px;align-items:center}
  .col{display:flex;flex-direction:column;gap:8px}
  button{padding:8px 10px;border-radius:6px;border:1px solid #ddd;background:#fafafa;cursor:pointer}
  input,select{padding:8px;border-radius:6px;border:1px solid #ccc}
  .hidden{display:none}
  .card-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(90px,1fr));gap:8px}
  .card{background:var(--card);border:1px solid #ddd;border-radius:6px;padding:6px;text-align:center;font-size:12px}
  .card img{width:100%;height:96px;object-fit:cover;border-radius:4px}
  .small{font-size:12px;color:#666}
  .controls{display:flex;gap:8px;flex-wrap:wrap}
  .timer{font-weight:700;color:#c33}
  .status{padding:8px;background:#f2f8ff;border-radius:6px;border:1px solid #dfefff}
  textarea{width:100%;min-height:64px}
  .selected{outline:3px solid rgba(43,109,246,.18)}
  .hit{background:#eaf5ff}
  footer{font-size:12px;color:#666;margin-top:12px}
  .btn-primary{background:var(--accent);color:#fff;border:none}
  /* admin-login pane */
  #adminLogin{padding:12px;border:1px solid #eee;background:#fff;border-radius:8px}
</style>
</head>
<body>
<header><h1 style="display:inline">オセロニアポーカー — 単一ファイル完全版</h1></header>
<div class="wrap">

  <!-- スタート / ルーム作成 -->
  <div id="startView">
    <div class="row">
      <div class="col" style="flex:1">
        <label>合言葉（4桁・新規作成）<input id="newCode" maxlength="4" placeholder="例: 1234"></label>
        <label>人数上限<select id="maxPlayers"><option>4</option><option>5</option><option>6</option></select></label>
        <label>あなたのニックネーム<input id="creatorName" value="Player"></label>
        <div class="controls"><button id="btnCreate" class="btn-primary">部屋を作る</button>
        <button id="btnJoinExisting">合言葉で入室</button>
        <button id="btnAdminLogin">管理者ログイン</button></div>
        <div id="startMsg" class="small"></div>
      </div>

      <div style="width:320px">
        <div class="status"><strong>操作説明</strong>
          <div class="small">このファイルはサーバ不要の単一ファイル版です。複数端末で同時プレイするには別途サーバが必要ですが、同一ブラウザ内で「複数プレイヤーを作って」動かすことで全機能を試せます。<br>操作は「Act as」で操作するプレイヤーを切り替えて進めてください。</div>
        </div>
      </div>
    </div>
  </div>

  <!-- Join dialog -->
  <div id="joinView" class="hidden">
    <div class="row">
      <div class="col" style="flex:1">
        <label>合言葉（4桁）<input id="joinCode" maxlength="4"></label>
        <label>ニックネーム<input id="joinName" value="Player"></label>
        <div class="controls">
          <button id="btnJoin" class="btn-primary">入室</button>
          <button id="btnBackStart">戻る</button>
        </div>
        <div id="joinMsg" class="small"></div>
      </div>
    </div>
  </div>

  <!-- Room Lobby -->
  <div id="lobbyView" class="hidden">
    <div class="row">
      <div style="flex:1">
        <div><strong>ルーム</strong> <span id="roomCodeDisplay"></span></div>
        <div class="small">部屋主: <span id="roomOwner"></span></div>
        <div style="margin-top:8px">
          <label>Act as:
            <select id="actAsSelect"></select>
          </label>
          <label style="margin-left:8px">自分の表示名<input id="myDisplay" style="width:160px"></label>
        </div>
        <div style="margin-top:8px">
          <button id="btnReady">準備OK</button>
          <button id="btnStartGame" class="hidden btn-primary">ゲーム開始（部屋主のみ）</button>
          <button id="btnLeave">退出</button>
          <button id="btnDissolve" class="hidden">部屋解散（部屋主）</button>
        </div>
        <div style="margin-top:8px"><strong>プレイヤー一覧</strong></div>
        <div id="playersList" class="card-grid" style="margin-top:8px"></div>
      </div>
      <div style="width:340px">
        <div class="status">
          <div><strong>ルーム操作</strong></div>
          <div class="small">部屋主はキック・解散権限を持ちます。合言葉は重複禁止です（同一ブラウザ内ルームとして管理）。</div>
        </div>
        <div style="margin-top:12px">
          <strong>管理者</strong>
          <div class="small">管理者ログインでカード編集が可能。</div>
          <button id="btnOpenAdmin">管理画面を開く</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Game area -->
  <div id="gameView" class="hidden">
    <div class="row" style="align-items:flex-start">
      <div style="flex:1">
        <div style="display:flex;align-items:center;gap:12px">
          <strong>状態:</strong> <div id="phaseLabel" class="small hit">待機</div>
          <div class="small">順番表示: <span id="orderDisplay"></span></div>
          <div class="small">現在ターン: <span id="currentTurnDisplay"></span></div>
          <div style="margin-left:auto"><button id="btnBackLobby">ロビーへ戻る</button></div>
        </div>

        <!-- Hand area -->
        <h3>あなたの手札（Act asに合わせて表示）</h3>
        <div id="handArea" class="card-grid"></div>

        <!-- Exchange area -->
        <div id="exchangePanel" class="hidden">
          <h4>交換フェーズ（順番制）</h4>
          <div>ラウンド: <span id="exchangeRoundDisplay">1</span> / <span id="exchangeIndexDisplay">0</span></div>
          <div>残り時間: <span id="exchangeTimer" class="timer">30</span></div>
          <div class="controls">
            <button id="btnExchangeDo">選択したカードを交換</button>
            <button id="btnExchangeOK">OK（交換なし）</button>
          </div>
          <div class="small">※捨てたカードは山札の一番下へ入ります</div>
        </div>

        <!-- Submit area -->
        <div id="submitPanel" class="hidden">
          <h4>提出フェーズ（全員同時）</h4>
          <div>残り時間: <span id="submitTimer" class="timer">100</span></div>
          <div class="small">カードをドラッグで並べ替え、役名を入力して提出</div>
          <div id="submitHand" class="card-grid" style="margin-top:8px"></div>
          <div style="margin-top:8px">
            役名: <input id="roleInput" style="width:60%"><button id="btnSubmit" class="btn-primary">提出</button>
          </div>
        </div>

        <!-- Vote area -->
        <div id="votePanel" class="hidden">
          <h4>投票フェーズ</h4>
          <div>残り時間: <span id="voteTimer" class="timer">120</span></div>
          <div id="submissionsArea" style="margin-top:8px"></div>
        </div>

        <!-- Result area -->
        <div id="resultPanel" class="hidden">
          <h4>結果</h4>
          <div id="resultDisplay"></div>
          <div style="margin-top:8px"><button id="btnNextGame">次のゲーム（リセット）</button></div>
        </div>
      </div>

      <!-- Right column: deck / discard / log -->
      <div style="width:320px">
        <div style="margin-bottom:8px"><strong>山札</strong> <span id="deckCount"></span> / <strong>捨て札</strong> <span id="discardCount"></span></div>
        <div style="margin-bottom:8px">
          <button id="btnShowDeck">山札の中身（デバッグ）</button>
        </div>
        <div style="margin-top:12px">
          <strong>操作ログ</strong>
          <div id="logArea" style="height:320px;overflow:auto;border:1px solid #eee;padding:8px;background:#fafafa"></div>
        </div>
      </div>
    </div>
  </div>

  <!-- Admin panel -->
  <div id="adminPanel" class="hidden" style="margin-top:12px">
    <h3>管理画面 — カード編集</h3>
    <div class="small">カードはローカル（ブラウザの LocalStorage）に保存されます。画像はアップロードすると DataURL として保存され、即時ゲームに反映されます。</div>
    <div style="margin-top:8px">
      <label>カード検索（ID or 名前）<input id="adminSearch" placeholder="例: 001"></label>
    </div>
    <div id="adminCards" class="card-grid" style="margin-top:8px;height:420px;overflow:auto"></div>
    <div style="margin-top:8px">
      <button id="btnExport">カードデータをエクスポート（JSON）</button>
      <button id="btnImport">カードデータをインポート（JSON）</button>
      <input type="file" id="importFile" accept="application/json" class="hidden">
    </div>
  </div>

  <!-- Admin login pane (inline form) -->
  <div id="adminLogin" class="hidden" style="margin-top:12px">
    <h3>管理者ログイン</h3>
    <div class="small">管理者操作をするにはパスワードが必要です（ローカルファイルのため本番用の扱いにはご注意ください）</div>
    <div style="margin-top:8px">
      <input id="adminPassword" type="password" placeholder="パスワード">
      <button id="adminLoginBtn" class="btn-primary">ログイン</button>
      <button id="adminLoginCancel">キャンセル</button>
    </div>
    <div id="adminLoginMsg" class="small"></div>
  </div>

  <footer style="margin-top:12px" class="small">※このファイルはローカルでデータを保持します。ブラウザを変えるとデータは引き継がれません。複数端末で同時プレイさせたい場合はサーバ版を導入します。</footer>
</div>

<script>
/* -----------------------
   定数・初期データ
   ----------------------- */
const CARD_TOTAL = 90;
const ADMIN_PASSWORD = 'nanamiya333';

/* -----------------------
   DB: localStorage 操作
   ----------------------- */
function defaultCards(){ const arr=[]; for(let i=1;i<=CARD_TOTAL;i++){const id=String(i).padStart(3,'0'); arr.push({id,name:`Card ${id}`,img:null});} return arr; }
function loadCardsFromStorage(){ const raw = localStorage.getItem('op_cards'); if(!raw) return defaultCards(); try{ return JSON.parse(raw);}catch(e){return defaultCards();} }
function saveCardsToStorage(arr){ localStorage.setItem('op_cards', JSON.stringify(arr)); }
function loadRoomsFromStorage(){ const raw = localStorage.getItem('op_rooms'); if(!raw) return {}; try{ return JSON.parse(raw);}catch(e){return {};}}
function saveRoomsToStorage(r){ localStorage.setItem('op_rooms', JSON.stringify(r)); }

/* -----------------------
   グローバル状態
   ----------------------- */
let cards = loadCardsFromStorage();
let rooms = loadRoomsFromStorage();
let currentRoom = null;
let actingPlayerId = null;

/* ユーティリティ */
function log(msg){ const el=document.getElementById('logArea'); el.innerHTML = `<div class="small">[${(new Date()).toLocaleTimeString()}] ${escapeHtml(msg)}</div>` + el.innerHTML; }
function escapeHtml(s){ return (s+'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
function uid(prefix='p'){ return prefix + Math.random().toString(36).slice(2,9); }
function shuffle(arr){ for(let i=arr.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [arr[i],arr[j]]=[arr[j],arr[i]] } }

/* UI 初期化 */
document.getElementById('btnCreate').onclick = createRoom;
document.getElementById('btnJoinExisting').onclick = ()=>{ showView('joinView'); };
document.getElementById('btnBackStart').onclick = ()=> showView('startView');
document.getElementById('btnJoin').onclick = joinRoom;
document.getElementById('btnStartGame').onclick = startGameAsOwner;
document.getElementById('btnBackLobby').onclick = ()=> { showView('lobbyView'); };
document.getElementById('btnReady').onclick = toggleReady;
document.getElementById('btnLeave').onclick = leaveRoom;
document.getElementById('btnDissolve').onclick = dissolveRoom;
document.getElementById('btnOpenAdmin').onclick = ()=> showView('adminPanel');
document.getElementById('btnShowDeck').onclick = ()=>{ const r=currentRoomObj(); if(r) alert(JSON.stringify(r.deck.map(c=>c.id).slice(0,50))); else alert('no room'); };
document.getElementById('btnExchangeDo').onclick = doExchange;
document.getElementById('btnExchangeOK').onclick = exchangeOK;
document.getElementById('btnSubmit').onclick = submitWork;
document.getElementById('btnNextGame').onclick = resetAfterGame;
document.getElementById('btnExport').onclick = exportCards;
document.getElementById('btnImport').onclick = ()=> document.getElementById('importFile').click();
document.getElementById('importFile').onchange = importCardsFromFile;

/* admin login handlers */
document.getElementById('btnAdminLogin').addEventListener('click', ()=> showView('adminLogin'));
document.getElementById('adminLoginCancel').addEventListener('click', ()=> showView('startView'));
document.getElementById('adminLoginBtn').addEventListener('click', ()=>{
  const pass = document.getElementById('adminPassword').value || '';
  if(pass === ADMIN_PASSWORD){
    showView('adminPanel');
    renderAdminCards();
    document.getElementById('adminPassword').value = '';
  } else {
    document.getElementById('adminLoginMsg').innerText = 'パスワードが違います';
  }
});

/* show simple view */
function showView(id){
  const views = ['startView','joinView','lobbyView','gameView','adminPanel','adminLogin'];
  views.forEach(v=>{ const el=document.getElementById(v); if(el) el.classList.add('hidden'); });
  const target = document.getElementById(id);
  if(target) target.classList.remove('hidden');
}

/* Room 管理 */
function createRoom(){
  const code = (document.getElementById('newCode').value || '').trim();
  const max = parseInt(document.getElementById('maxPlayers').value||4);
  const creator = document.getElementById('creatorName').value || 'Owner';
  if(!/^\d{4}$/.test(code)){ alert('合言葉は4桁の数字にしてください'); return; }
  if(rooms[code]){ alert('その合言葉は既に使われています'); return; }
  const room = { code, owner:null, maxPlayers: Math.min(6, Math.max(4,max)), players:[], state:'waiting', deck:[], discard:[], turnOrder:[], exchangeRound:1, exchangeIndex:0, submitted:{}, votes:{}, voteCounts:{}, phase:'waiting' };
  rooms[code] = room; saveRoomsToStorage(rooms);
  joinRoomWithName(code, creator, true);
  document.getElementById('startMsg').innerText = `部屋 ${code} を作成しました。`;
}
function joinRoom(){
  const code = document.getElementById('joinCode').value.trim();
  const name = document.getElementById('joinName').value || ('P'+Math.floor(Math.random()*1000));
  if(!/^\d{4}$/.test(code)){ document.getElementById('joinMsg').innerText = '合言葉は4桁'; return; }
  if(!rooms[code]){ document.getElementById('joinMsg').innerText = 'その合言葉の部屋はありません'; return; }
  joinRoomWithName(code, name, false);
}
function joinRoomWithName(code, name, isCreator){
  const room = rooms[code];
  if(room.players.length >= room.maxPlayers){ alert('満員です'); return; }
  const pid = uid('p');
  const player = {id:pid, nick: name, ready:false, hand:[], score:0, connected:true};
  room.players.push(player);
  if(isCreator){ room.owner = pid; }
  saveRoomsToStorage(rooms);
  currentRoom = code;
  actingPlayerId = pid;
  document.getElementById('myDisplay').value = player.nick;
  refreshLobby();
  showView('lobbyView');
  log(`${player.nick} が部屋 ${code} に参加しました`);
}
function refreshLobby(){
  const room = currentRoomObj(); if(!room) return;
  document.getElementById('roomCodeDisplay').innerText = room.code;
  document.getElementById('roomOwner').innerText = (room.owner? (room.players.find(p=>p.id===room.owner)||{}).nick : '');
  const wrap = document.getElementById('playersList'); wrap.innerHTML = '';
  room.players.forEach(p=>{
    const div = document.createElement('div'); div.className='card';
    div.innerHTML = `<div style="font-weight:700">${escapeHtml(p.nick)}</div><div class="small">ready: ${p.ready? '✅': '—'}</div>`;
    if(room.owner === actingPlayerId && p.id !== room.owner){
      const b = document.createElement('button'); b.textContent='Kick'; b.onclick = ()=> { kickPlayer(p.id); };
      div.appendChild(b);
    }
    wrap.appendChild(div);
  });
  const sel = document.getElementById('actAsSelect'); sel.innerHTML = '';
  room.players.forEach(p => { const o = document.createElement('option'); o.value = p.id; o.text = p.nick; sel.appendChild(o); });
  sel.value = actingPlayerId || (room.players[0] && room.players[0].id) || '';
  sel.onchange = ()=> { actingPlayerId = sel.value; renderHandArea(); };
  const allReady = room.players.length >= 4 && room.players.every(p=>p.ready);
  document.getElementById('btnStartGame').classList.toggle('hidden', !(allReady && actingPlayerId===room.owner));
  document.getElementById('btnDissolve').classList.toggle('hidden', actingPlayerId===room.owner? false : true);
}
function currentRoomObj(){ return currentRoom ? rooms[currentRoom] : null; }
function leaveRoom(){ const room=currentRoomObj(); if(!room) return; const idx=room.players.findIndex(p=>p.id===actingPlayerId); if(idx!==-1){ const name=room.players[idx].nick; room.players.splice(idx,1); log(`${name} が退出しました`);} if(room.owner===actingPlayerId){ if(room.players.length>0) room.owner=room.players[0].id; else delete rooms[room.code]; } saveRoomsToStorage(rooms); currentRoom=null; actingPlayerId=null; showView('startView'); }
function dissolveRoom(){ const room=currentRoomObj(); if(!room) return; if(room.owner!==actingPlayerId){ alert('権限なし'); return; } if(!confirm('本当に部屋を解散しますか？')) return; delete rooms[room.code]; saveRoomsToStorage(rooms); currentRoom=null; actingPlayerId=null; showView('startView'); log('部屋を解散しました'); }
function kickPlayer(targetId){ const room=currentRoomObj(); if(!room) return; if(room.owner!==actingPlayerId){ alert('権限なし'); return; } const idx=room.players.findIndex(p=>p.id===targetId); if(idx===-1) return; const name=room.players[idx].nick; room.players.splice(idx,1); saveRoomsToStorage(rooms); refreshLobby(); log(`${name} をキックしました`); }
function toggleReady(){ const room=currentRoomObj(); if(!room) return; const player=room.players.find(p=>p.id===actingPlayerId); if(!player) return; player.ready=!player.ready; saveRoomsToStorage(rooms); refreshLobby(); log(`${player.nick} の準備状態: ${player.ready}`); }

/* GAME START */
function startGameAsOwner(){ const room=currentRoomObj(); if(!room) return; if(room.owner!==actingPlayerId){ alert('部屋主のみ開始可能'); return; } if(room.players.length<4 || !room.players.every(p=>p.ready)){ alert('参加者が揃っていないか準備ができていません'); return; } const deck=loadCardsFromStorage().map(c=>({id:c.id,name:c.name,img:c.img})); shuffle(deck); room.deck=deck; room.discard=[]; room.state='playing'; const order=room.players.map(p=>p.id); shuffle(order); room.turnOrder=order; room.exchangeRound=1; room.exchangeIndex=0; for(const pid of order){ const p=room.players.find(x=>x.id===pid); p.hand=room.deck.splice(0,5); p.score=0; } room.submitted={}; room.votes={}; room.voteCounts={}; room.phase='exchange'; saveRoomsToStorage(rooms); showView('gameView'); refreshGameUI(); log('ゲーム開始: 順番 ' + room.turnOrder.map(id=>room.players.find(p=>p.id===id).nick).join(', ')); startExchangeTurnLoop(); }

/* Exchange phase */
let exchangeTimerInterval = null;
function startExchangeTurnLoop(){ const room=currentRoomObj(); if(!room) return; if(room.exchangeRound>2){ startSubmitPhase(); return; } if(room.exchangeIndex>=room.turnOrder.length){ room.exchangeRound++; room.exchangeIndex=0; saveRoomsToStorage(rooms); startExchangeTurnLoop(); return; } const pid=room.turnOrder[room.exchangeIndex]; room.phase='exchange'; saveRoomsToStorage(rooms); refreshGameUI(); let timeLeft=30; document.getElementById('exchangeTimer').innerText=timeLeft; if(exchangeTimerInterval){ clearInterval(exchangeTimerInterval); exchangeTimerInterval=null; } exchangeTimerInterval=setInterval(()=>{ timeLeft--; document.getElementById('exchangeTimer').innerText=timeLeft; if(timeLeft<=0){ clearInterval(exchangeTimerInterval); exchangeTimerInterval=null; log(`${(room.players.find(p=>p.id===pid)||{}).nick} の交換タイムアウト`); room.exchangeIndex++; saveRoomsToStorage(rooms); startExchangeTurnLoop(); } },1000); }
function renderHandArea(){ const room=currentRoomObj(); if(!room) return; const player=room.players.find(p=>p.id===actingPlayerId); const wrap=document.getElementById('handArea'); wrap.innerHTML=''; if(!player) return; player.hand.forEach((c,idx)=>{ const div=document.createElement('div'); div.className='card'; div.dataset.index=idx; const img=document.createElement('img'); img.src=c.img||placeholderForId(c.id); const nm=document.createElement('div'); nm.textContent=c.name; div.appendChild(img); div.appendChild(nm); div.onclick=()=>{ div.classList.toggle('selected'); }; wrap.appendChild(div); }); renderSubmitHand(); }
function placeholderForId(id){ const svg=`<svg xmlns='http://www.w3.org/2000/svg' width='200' height='300'><rect width='100%' height='100%' fill='#eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' fill='#666' font-size='18'>${id}</text></svg>`; return 'data:image/svg+xml;base64,'+btoa(svg); }
function doExchange(){ const room=currentRoomObj(); if(!room) return; const pid=room.turnOrder[room.exchangeIndex]; if(pid!==actingPlayerId){ alert('今はこのプレイヤーの番ではありません。Act as を切り替えてください'); return; } const player=room.players.find(p=>p.id===pid); const selEls=Array.from(document.getElementById('handArea').querySelectorAll('.selected')); if(selEls.length===0){ alert('交換したいカードを選択してください（0-5枚）'); return; } const idxs=selEls.map(el=>parseInt(el.dataset.index)).sort((a,b)=>b-a); const discarded=[]; for(const idx of idxs){ if(idx>=0 && idx<player.hand.length) discarded.push(player.hand.splice(idx,1)[0]); } room.deck.push(...discarded); const newcards=room.deck.splice(0,discarded.length); player.hand.push(...newcards); if(exchangeTimerInterval){ clearInterval(exchangeTimerInterval); exchangeTimerInterval=null; } room.exchangeIndex++; saveRoomsToStorage(rooms); renderHandArea(); refreshGameUI(); log(`${player.nick} が ${discarded.length} 枚交換しました`); startExchangeTurnLoop(); }
function exchangeOK(){ const room=currentRoomObj(); if(!room) return; const pid=room.turnOrder[room.exchangeIndex]; if(pid!==actingPlayerId){ alert('今はこのプレイヤーの番ではありません'); return; } if(exchangeTimerInterval){ clearInterval(exchangeTimerInterval); exchangeTimerInterval=null; } room.exchangeIndex++; saveRoomsToStorage(rooms); log(`${(room.players.find(p=>p.id===pid)||{}).nick} がOKしました`); startExchangeTurnLoop(); }

/* Submit phase */
let submitInterval=null;
function startSubmitPhase(){ const room=currentRoomObj(); if(!room) return; room.phase='submit'; room.submitTimeLeft=100; for(const pid of room.turnOrder){ if(!room.submitted[pid]){ const p=room.players.find(x=>x.id===pid); room.submitted[pid]={order:p.hand.map(c=>c.id), roleName:'', submitted:false}; } } saveRoomsToStorage(rooms); refreshGameUI(); let t=room.submitTimeLeft; document.getElementById('submitTimer').innerText=t; if(submitInterval){ clearInterval(submitInterval); submitInterval=null; } submitInterval=setInterval(()=>{ t--; document.getElementById('submitTimer').innerText=t; const all = room.turnOrder.every(pid=>!!room.submitted[pid] && room.submitted[pid].submitted); if(all){ clearInterval(submitInterval); submitInterval=null; startVotePhase(); } if(t<=0){ clearInterval(submitInterval); submitInterval=null; for(const pid of room.turnOrder){ if(!room.submitted[pid] || !room.submitted[pid].submitted){ const p=room.players.find(x=>x.id===pid); room.submitted[pid]={order:p.hand.map(c=>c.id), roleName:'', submitted:true}; } } saveRoomsToStorage(rooms); startVotePhase(); } },1000); }
function renderSubmitHand(){ const container=document.getElementById('submitHand'); container.innerHTML=''; const room=currentRoomObj(); if(!room) return; const player=room.players.find(p=>p.id===actingPlayerId); if(!player) return; player.hand.forEach((c,idx)=>{ const div=document.createElement('div'); div.className='card'; div.dataset.id=c.id; const img=document.createElement('img'); img.src=c.img||placeholderForId(c.id); const nm=document.createElement('div'); nm.textContent=c.name; div.appendChild(img); div.appendChild(nm); container.appendChild(div); }); makeSortable(container); }
function makeSortable(container){ let dragEl=null; Array.from(container.children).forEach(ch=>{ ch.draggable=true; ch.ondragstart=(e)=>{ dragEl=ch; e.dataTransfer.setData('text/plain',''); ch.style.opacity=.4; }; ch.ondragend=()=>{ if(dragEl) dragEl.style.opacity=1; dragEl=null; }; ch.ondragover=(e)=>{ e.preventDefault(); }; ch.ondrop=(e)=>{ e.preventDefault(); if(!dragEl || dragEl===ch) return; container.insertBefore(dragEl, ch.nextSibling); }; }); }
function submitWork(){ const room=currentRoomObj(); if(!room) return; if(room.phase!=='submit'){ alert('今は提出フェーズではありません'); return; } const nodes=Array.from(document.getElementById('submitHand').children); if(nodes.length!==5){ alert('5枚を並べてください'); return; } const order=nodes.map(n=>n.dataset.id); const roleName=document.getElementById('roleInput').value.trim(); room.submitted[actingPlayerId]={order, roleName, submitted:true}; saveRoomsToStorage(rooms); log(`${(room.players.find(p=>p.id===actingPlayerId)||{}).nick} が提出しました: ${roleName || '(無題)'}`); }

/* Vote phase */
let voteInterval=null;
function startVotePhase(){ const room=currentRoomObj(); if(!room) return; room.phase='vote'; room.voteTimeLeft=120; room.votes={}; room.voteCounts={}; saveRoomsToStorage(rooms); refreshGameUI(); renderSubmissions(); let t=room.voteTimeLeft; document.getElementById('voteTimer').innerText=t; if(voteInterval){ clearInterval(voteInterval); voteInterval=null; } voteInterval=setInterval(()=>{ t--; document.getElementById('voteTimer').innerText=t; const allVoted = room.turnOrder.every(pid => Object.values(room.votes).includes(pid)); if(allVoted){ clearInterval(voteInterval); voteInterval=null; finishVote(); } if(t<=0){ clearInterval(voteInterval); voteInterval=null; finishVote(); } },1000); }
function renderSubmissions(){ const room=currentRoomObj(); if(!room) return; const container=document.getElementById('submissionsArea'); container.innerHTML=''; for(const pid of room.turnOrder){ const sub = room.submitted[pid] || {order: room.players.find(p=>p.id===pid).hand.map(c=>c.id), roleName:''}; const box=document.createElement('div'); box.className='card'; box.innerHTML=`<div style="font-weight:700">${escapeHtml((room.players.find(p=>p.id===pid)||{}).nick||pid)}</div><div class="small">${escapeHtml(sub.roleName)}</div>`; const row=document.createElement('div'); row.style.display='flex'; row.style.gap='6px'; row.style.marginTop='6px'; sub.order.forEach(cid=>{ const cc=cards.find(x=>x.id===cid) || {id:cid,name:cid,img:null}; const thumb=document.createElement('img'); thumb.src=cc.img||placeholderForId(cc.id); thumb.style.width='46px'; thumb.style.height='68px'; thumb.style.objectFit='cover'; row.appendChild(thumb); }); box.appendChild(row); if(actingPlayerId && actingPlayerId!==pid && !(room.votes && room.votes[actingPlayerId])){ const voteBtn=document.createElement('button'); voteBtn.textContent='投票'; voteBtn.onclick=()=>{ room.votes[actingPlayerId]=pid; room.voteCounts[pid]=(room.voteCounts[pid]||0)+1; saveRoomsToStorage(rooms); log(`${(room.players.find(p=>p.id===actingPlayerId)||{}).nick} が ${(room.players.find(x=>x.id===pid)||{}).nick} に投票しました`); renderSubmissions(); }; box.appendChild(voteBtn); } else { const note=document.createElement('div'); note.className='small'; note.innerText = actingPlayerId===pid ? 'あなたの提出' : ((room.votes && room.votes[actingPlayerId]) ? '投票済' : ''); box.appendChild(note); } container.appendChild(box); } }
function finishVote(){ const room=currentRoomObj(); if(!room) return; for(const pid of room.turnOrder) room.voteCounts[pid]=room.voteCounts[pid]||0; let max=-1; for(const pid of room.turnOrder) if(room.voteCounts[pid]>max) max=room.voteCounts[pid]; const winners=room.turnOrder.filter(pid=>room.voteCounts[pid]===max); document.getElementById('resultPanel').classList.remove('hidden'); let html='<div class="small">得票数</div>'; for(const pid of room.turnOrder){ html+=`<div>${escapeHtml((room.players.find(p=>p.id===pid)||{}).nick||pid)}: ${room.voteCounts[pid]}</div>`; } html+=`<div style="margin-top:8px;"><strong>勝者: ${winners.map(w=>escapeHtml((room.players.find(p=>p.id===w)||{}).nick||w)).join(', ')}</strong></div>`; document.getElementById('resultDisplay').innerHTML=html; log('投票終了。結果表示'); room.state='waiting'; room.phase='waiting'; saveRoomsToStorage(rooms); refreshGameUI(); }
function resetAfterGame(){ const room=currentRoomObj(); if(!room) return; for(const p of room.players){ p.hand=[]; p.ready=false; p.score=0; } room.deck=[]; room.discard=[]; room.turnOrder=[]; room.exchangeRound=1; room.exchangeIndex=0; room.submitted={}; room.votes={}; room.voteCounts={}; room.phase='waiting'; saveRoomsToStorage(rooms); document.getElementById('resultPanel').classList.add('hidden'); showView('lobbyView'); refreshLobby(); log('次のゲームのためにリセットしました'); }

/* Admin */
function renderAdminCards(){ const wrap=document.getElementById('adminCards'); wrap.innerHTML=''; const search=(document.getElementById('adminSearch')||{value:''}).value.toLowerCase(); for(const c of cards){ if(search && !(c.id.includes(search) || c.name.toLowerCase().includes(search))) continue; const box=document.createElement('div'); box.className='card'; const img=document.createElement('img'); img.src=c.img||placeholderForId(c.id); img.style.height='90px'; const name=document.createElement('input'); name.value=c.name; name.onchange=()=>{ c.name=name.value; saveCardsToStorage(cards); renderAdminCards(); }; const file=document.createElement('input'); file.type='file'; file.accept='image/*'; file.onchange=(ev)=>{ const f=ev.target.files[0]; const r=new FileReader(); r.onload=(e)=>{ c.img=e.target.result; saveCardsToStorage(cards); renderAdminCards(); }; if(f) r.readAsDataURL(f); }; box.appendChild(img); box.appendChild(name); box.appendChild(file); wrap.appendChild(box); } }
function exportCards(){ const data=JSON.stringify(cards); const blob=new Blob([data],{type:'application/json'}); const url=URL.createObjectURL(blob); const a=document.createElement('a'); a.href=url; a.download='cards.json'; a.click(); URL.revokeObjectURL(url); }
function importCardsFromFile(e){ const f=e.target.files[0]; if(!f) return; const r=new FileReader(); r.onload=(ev)=>{ try{ const parsed=JSON.parse(ev.target.result); if(Array.isArray(parsed)&&parsed.length>0&&parsed[0].id){ cards=parsed; saveCardsToStorage(cards); renderAdminCards(); alert('インポート完了'); } else alert('不正なファイル'); }catch(err){ alert('読み込み失敗'); } }; r.readAsText(f); }

/* init */
(function init(){ const stored=localStorage.getItem('op_cards'); if(!stored){ saveCardsToStorage(cards); } else { cards=loadCardsFromStorage(); } const storedRooms=localStorage.getItem('op_rooms'); if(storedRooms) rooms=loadRoomsFromStorage(); showView('startView'); })();
function refreshGameUI(){ const room=currentRoomObj(); if(!room) return; document.getElementById('phaseLabel').innerText=room.phase||'playing'; document.getElementById('orderDisplay').innerText=(room.turnOrder||[]).map(id=>(room.players.find(p=>p.id===id)||{}).nick).join(' → '); document.getElementById('currentTurnDisplay').innerText=(room.turnOrder&&room.turnOrder[room.exchangeIndex])?(room.players.find(p=>p.id===room.turnOrder[room.exchangeIndex])||{}).nick:'—'; document.getElementById('deckCount').innerText=room.deck.length; document.getElementById('discardCount').innerText=room.discard.length; document.getElementById('exchangeRoundDisplay').innerText=room.exchangeRound; document.getElementById('exchangeIndexDisplay').innerText=(room.exchangeIndex+1)+'/'+room.turnOrder.length; document.getElementById('exchangePanel').classList.toggle('hidden', room.phase!=='exchange'); document.getElementById('submitPanel').classList.toggle('hidden', room.phase!=='submit'); document.getElementById('votePanel').classList.toggle('hidden', room.phase!=='vote'); renderHandArea(); }
</script>
</body>
</html>
