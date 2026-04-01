// ── CONFIG ──────────────────────────────────────────────────────────────────
const SUPABASE_URL  = 'https://vwhkitpaomguuanwbdxy.supabase.co';
const SUPABASE_ANON = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InZ3aGtpdHBhb21ndXVhbndiZHh5Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM0MzQ3NDksImV4cCI6MjA4OTAxMDc0OX0.13MqBoAdjCZb_nw01leSV3XCRk0vqKeTe1L_QFaZ93M';
const ROOM_ID       = 1;
const HEARTBEAT_MS  = 20_000;   // ping presence every 20s
const STALE_MS      = 60_000;   // players absent >60s are pruned

// ── INIT ─────────────────────────────────────────────────────────────────────
const sb = supabase.createClient(SUPABASE_URL, SUPABASE_ANON);

let myName     = null;
let myId       = null;
let heartbeat  = null;
let msgChannel = null;
let presChannel= null;

// ── DOM ───────────────────────────────────────────────────────────────────────
const loginScreen = document.getElementById('login-screen');
const nameInput   = document.getElementById('name-input');
const loginError  = document.getElementById('login-error');
const output      = document.getElementById('output');
const cmdInput    = document.getElementById('cmd-input');
const playerList  = document.getElementById('player-list');
const playerCount = document.getElementById('player-count');
const roomDesc    = document.getElementById('room-desc');

// ── PRINT ─────────────────────────────────────────────────────────────────────
function timestamp() {
  const d = new Date();
  return d.getHours().toString().padStart(2,'0') + ':' +
         d.getMinutes().toString().padStart(2,'0') + ':' +
         d.getSeconds().toString().padStart(2,'0');
}

function print(type, name, body) {
  const div  = document.createElement('div');
  div.className = 'msg ' + type + (name === myName ? ' you' : '');

  const ts   = document.createElement('span');
  ts.className = 'ts';
  ts.textContent = timestamp();

  const bodySpan = document.createElement('span');
  bodySpan.className = 'body';

  if (type === 'say') {
    const nameSpan = document.createElement('span');
    nameSpan.className = 'name';
    nameSpan.textContent = name + ': ';
    div.appendChild(ts);
    div.appendChild(nameSpan);
    bodySpan.textContent = body;
  } else if (type === 'emote') {
    bodySpan.textContent = '* ' + name + ' ' + body;
    div.appendChild(ts);
  } else {
    bodySpan.textContent = body;
    div.appendChild(ts);
  }

  div.appendChild(bodySpan);
  output.appendChild(div);
  output.scrollTop = output.scrollHeight;
}

// ── LOGIN ─────────────────────────────────────────────────────────────────────
nameInput.addEventListener('keydown', async (e) => {
  if (e.key !== 'Enter') return;
  const raw = nameInput.value.trim();
  if (!raw) return;

  // Sanitise: letters, numbers, underscore, dash only
  const name = raw.replace(/[^a-zA-Z0-9_\-]/g, '').slice(0, 24);
  if (!name) {
    loginError.textContent = 'Name must contain letters or numbers.';
    return;
  }

  loginError.textContent = 'Connecting...';

  // Upsert player (name is unique — if taken it will conflict)
  const { data, error } = await sb
    .from('players')
    .upsert({ name, last_seen: new Date().toISOString() }, { onConflict: 'name' })
    .select()
    .single();

  if (error) {
    loginError.textContent = 'Name taken or DB error: ' + error.message;
    return;
  }

  myName = data.name;
  myId   = data.id;

  loginScreen.style.display = 'none';
  cmdInput.disabled = false;
  cmdInput.focus();

  await startGame();
});

// ── START GAME ────────────────────────────────────────────────────────────────
async function startGame() {
  await loadRoom();
  await loadHistory();
  await refreshPlayers();

  subscribeMessages();
  subscribePlayers();
  startHeartbeat();

  await postSystem(`${myName} has entered The Crossroads.`, 'join');
  print('system', null, `Connected as ${myName}. Type 'help' for commands.`);

  // Prune stale players on load
  await pruneStalePlayers();

  window.addEventListener('beforeunload', onLeave);
}

// ── ROOM ──────────────────────────────────────────────────────────────────────
async function loadRoom() {
  const { data } = await sb.from('rooms').select('*').eq('id', ROOM_ID).single();
  if (data) {
    roomDesc.textContent = data.description;
  }
}

// ── HISTORY ───────────────────────────────────────────────────────────────────
async function loadHistory() {
  const { data } = await sb
    .from('messages')
    .select('*')
    .order('created_at', { ascending: false })
    .limit(40);

  if (!data) return;
  data.reverse().forEach(renderMessage);
}

function renderMessage(msg) {
  if (msg.type === 'say')    print('say',    msg.player_name, msg.body);
  else if (msg.type === 'emote')  print('emote', msg.player_name, msg.body);
  else if (msg.type === 'join')   print('join',  null, msg.body);
  else if (msg.type === 'leave')  print('leave', null, msg.body);
  else if (msg.type === 'system') print('system', null, msg.body);
}

// ── PLAYERS ───────────────────────────────────────────────────────────────────
async function refreshPlayers() {
  const cutoff = new Date(Date.now() - STALE_MS).toISOString();
  const { data } = await sb
    .from('players')
    .select('name, last_seen')
    .gte('last_seen', cutoff)
    .order('name');

  playerList.innerHTML = '';
  const players = data || [];

  players.forEach(p => {
    const div = document.createElement('div');
    div.className = 'player-entry' + (p.name === myName ? ' you' : '');
    div.textContent = p.name + (p.name === myName ? ' (you)' : '');
    playerList.appendChild(div);
  });

  const n = players.length;
  playerCount.textContent = `● ${n} present`;
}

async function pruneStalePlayers() {
  const cutoff = new Date(Date.now() - STALE_MS).toISOString();
  await sb.from('players').delete().lt('last_seen', cutoff);
}

// ── REALTIME: MESSAGES ────────────────────────────────────────────────────────
function subscribeMessages() {
  msgChannel = sb
    .channel('messages-channel')
    .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, (payload) => {
      renderMessage(payload.new);
    })
    .subscribe();
}

// ── REALTIME: PLAYERS ─────────────────────────────────────────────────────────
function subscribePlayers() {
  presChannel = sb
    .channel('players-channel')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'players' }, () => {
      refreshPlayers();
    })
    .subscribe();
}

// ── HEARTBEAT ─────────────────────────────────────────────────────────────────
function startHeartbeat() {
  heartbeat = setInterval(async () => {
    await sb.from('players')
      .update({ last_seen: new Date().toISOString() })
      .eq('id', myId);
  }, HEARTBEAT_MS);
}

// ── COMMANDS ──────────────────────────────────────────────────────────────────
cmdInput.addEventListener('keydown', async (e) => {
  if (e.key !== 'Enter') return;
  const raw = cmdInput.value.trim();
  cmdInput.value = '';
  if (!raw) return;
  await handleCommand(raw);
});

async function handleCommand(raw) {
  const lower = raw.toLowerCase();
  const parts  = raw.split(' ');
  const verb   = parts[0].toLowerCase();
  const rest   = parts.slice(1).join(' ');

  if (verb === 'say' || verb === '"' || verb === "'") {
    const text = verb === 'say' ? rest : raw.slice(1).trim();
    if (!text) { print('error', null, 'Say what?'); return; }
    await postMessage('say', text);

  } else if (verb === 'emote' || verb === ':') {
    const action = verb === 'emote' ? rest : raw.slice(1).trim();
    if (!action) { print('error', null, 'Emote what?'); return; }
    await postMessage('emote', action);

  } else if (verb === 'look' || verb === 'l') {
    const { data } = await sb.from('rooms').select('*').eq('id', ROOM_ID).single();
    if (data) {
      print('look', null, `[ ${data.name} ]`);
      print('look', null, data.description);
      print('look', null, `Exits: none.`);
    }

  } else if (verb === 'who') {
    await refreshPlayers();
    const cutoff = new Date(Date.now() - STALE_MS).toISOString();
    const { data } = await sb.from('players').select('name').gte('last_seen', cutoff);
    const names = (data || []).map(p => p.name).join(', ');
    print('system', null, `Present: ${names || 'nobody'}`);

  } else if (verb === 'help' || verb === '?') {
    print('system', null, '─────────────────────────────');
    print('system', null, 'say <msg>        — speak aloud');
    print('system', null, '"<msg>           — shorthand for say');
    print('system', null, 'emote <action>   — describe an action');
    print('system', null, ':<action>        — shorthand for emote');
    print('system', null, 'look  (or l)     — examine the room');
    print('system', null, 'who              — list who is here');
    print('system', null, 'quit             — disconnect');
    print('system', null, '─────────────────────────────');

  } else if (verb === 'quit' || verb === 'exit' || verb === 'logout') {
    await onLeave();
    print('system', null, 'Goodbye.');
    cmdInput.disabled = true;

  } else {
    print('error', null, `Unknown command: "${verb}". Type 'help' for a list.`);
  }
}

// ── POST HELPERS ──────────────────────────────────────────────────────────────
async function postMessage(type, body) {
  await sb.from('messages').insert({ player_name: myName, type, body });
}

async function postSystem(body, type = 'system') {
  await sb.from('messages').insert({ player_name: null, type, body });
}

// ── LEAVE ─────────────────────────────────────────────────────────────────────
async function onLeave() {
  clearInterval(heartbeat);
  if (msgChannel)  sb.removeChannel(msgChannel);
  if (presChannel) sb.removeChannel(presChannel);
  await postSystem(`${myName} has left The Crossroads.`, 'leave');
  await sb.from('players').delete().eq('id', myId);
}
