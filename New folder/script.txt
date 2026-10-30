const WORDS = ("apple brave crane dream eagle flame grape house input joint knife lemon music night ocean piano queen river stone tiger uncle voice water young zebra beach chair dance earth field ghost heart juice light money noise paper quiet smile table world bread cloud drink flood glass horse image magic plant radio sugar train wheel shirt tooth sweet snake sleep score shore spice taste trust value watch youth angry blank candy daily empty fresh giant happy lucky merry noble party proud rapid sharp tired usual vivid whale yield brush climb drive grass honey laugh march north olive pearl quick round south think unity visit woman adult bench cabin delay enemy favor grain habit ivory jelly kayak").split(" ");
const ROWS = 6, COLS = 5;
const KEYS = ["qwertyuiop", "asdfghjkl", "+zxcvbnm-"];
const $board = document.getElementById("board"), $kb = document.getElementById("kb"), $msg = document.getElementById("msg");
let answer, row, col, grid, over, busy, keyState;

function build() {
  $board.innerHTML = "";
  grid = [];
  for (let r = 0; r < ROWS; r++) {
    const el = document.createElement("div");
    el.className = "row";
    const tiles = [];
    for (let c = 0; c < COLS; c++) {
      const t = document.createElement("div");
      t.className = "t";
      el.appendChild(t);
      tiles.push(t);
    }
    $board.appendChild(el);
    grid.push({ el, tiles });
  }
  $kb.innerHTML = "";
  KEYS.forEach(line => {
    const kr = document.createElement("div");
    kr.className = "kr";
    [...line].forEach(ch => {
      const b = document.createElement("button");
      b.className = "k";
      if (ch === "+") { b.textContent = "Enter"; b.dataset.k = "Enter"; b.classList.add("wide"); }
      else if (ch === "-") { b.textContent = "Del"; b.dataset.k = "Backspace"; b.classList.add("wide"); }
      else { b.textContent = ch; b.dataset.k = ch; }
      b.addEventListener("click", () => press(b.dataset.k));
      kr.appendChild(b);
    });
    $kb.appendChild(kr);
  });
}

function start() {
  answer = WORDS[Math.floor(Math.random() * WORDS.length)];
  row = 0; col = 0; over = false; busy = false; keyState = {};
  $msg.textContent = "";
  build();
}

function say(text, ms) {
  $msg.textContent = text;
  if (ms) setTimeout(() => { if ($msg.textContent === text) $msg.textContent = ""; }, ms);
}

function press(k) {
  if (over || busy) return;
  if (k === "Enter") return submit();
  if (k === "Backspace") {
    if (col > 0) { col--; const t = grid[row].tiles[col]; t.textContent = ""; t.classList.remove("fill"); }
    return;
  }
  if (/^[a-z]$/.test(k) && col < COLS) {
    const t = grid[row].tiles[col];
    t.textContent = k;
    t.classList.add("fill");
    col++;
  }
}

// Result: "ok" (right spot), "near" (in word, wrong spot), "no" (not in word). Handles repeated letters.
function evaluate(guess) {
  const res = Array(COLS).fill("no");
  const left = {};
  for (let i = 0; i < COLS; i++) {
    if (guess[i] === answer[i]) res[i] = "ok";
    else left[answer[i]] = (left[answer[i]] || 0) + 1;
  }
  for (let i = 0; i < COLS; i++) {
    if (res[i] === "ok") continue;
    if (left[guess[i]] > 0) { res[i] = "near"; left[guess[i]]--; }
  }
  return res;
}

function submit() {
  if (col < COLS) {
    grid[row].el.classList.add("shake");
    setTimeout(() => grid[row].el.classList.remove("shake"), 400);
    return say("Not enough letters", 1500);
  }
  const guess = grid[row].tiles.map(t => t.textContent).join("");
  const res = evaluate(guess);
  busy = true;
  const rank = { no: 1, near: 2, ok: 3 };
  grid[row].tiles.forEach((t, i) => {
    setTimeout(() => {
      t.classList.add("flip");
      setTimeout(() => t.classList.add(res[i]), 250);
    }, i * 250);
  });
  setTimeout(() => {
    for (let i = 0; i < COLS; i++) {
      const ch = guess[i];
      if (!keyState[ch] || rank[res[i]] > rank[keyState[ch]]) keyState[ch] = res[i];
      const kb = $kb.querySelector('[data-k="' + ch + '"]');
      kb.classList.remove("ok", "near", "no");
      kb.classList.add(keyState[ch]);
    }
    busy = false;
    if (guess === answer) {
      over = true;
      say("You got it in " + (row + 1) + (row === 0 ? " try!" : " tries!"));
    } else if (row === ROWS - 1) {
      over = true;
      say("The word was " + answer.toUpperCase());
    } else { row++; col = 0; }
  }, COLS * 250 + 300);
}

document.addEventListener("keydown", e => {
  if (e.ctrlKey || e.metaKey || e.altKey) return;
  const k = e.key.length === 1 ? e.key.toLowerCase() : e.key;
  if (k === "Enter" && document.activeElement && document.activeElement.tagName === "BUTTON") return;
  press(k);
});
document.getElementById("new").addEventListener("click", e => { start(); e.currentTarget.blur(); });
start();