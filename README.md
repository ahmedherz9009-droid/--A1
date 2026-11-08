<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>English A1 Game</title>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      padding: 20px;
      background: #f1f5f9;
      color: #172033;
      font-family: Arial, sans-serif;
    }
    .game {
      width: min(100%, 600px);
      padding: 28px;
      border-radius: 18px;
      background: white;
      box-shadow: 0 10px 35px #17203318;
    }
    h1 { margin: 0 0 8px; text-align: center; }
    .subtitle { margin: 0 0 22px; text-align: center; color: #64748b; }
    .meta { display: flex; justify-content: space-between; gap: 12px; color: #64748b; font-size: 14px; }
    .question {
      margin: 24px 0;
      text-align: center;
      direction: ltr;
      font-size: 23px;
      font-weight: bold;
    }
    .options { display: grid; gap: 10px; }
    button {
      border: 0;
      border-radius: 10px;
      padding: 14px;
      font-size: 17px;
      cursor: pointer;
    }
    .option { background: #e8eef8; color: #172033; direction: ltr; }
    .option:hover:not(:disabled) { background: #d8e4f7; }
    .option:disabled { cursor: default; }
    .correct { background: #bbf7d0 !important; }
    .wrong { background: #fecaca !important; }
    #feedback { min-height: 28px; margin: 18px 0 10px; text-align: center; font-weight: bold; }
    #next, #restart { width: 100%; background: #2563eb; color: white; }
    #next:disabled { background: #94a3b8; cursor: default; }
    .note { margin-top: 16px; color: #64748b; font-size: 13px; text-align: center; }
  </style>
</head>
<body>
  <main class="game">
    <h1>لعبة English A1</h1>
    <p class="subtitle">اختر الإجابة الصحيحة</p>

    <div class="meta">
      <span id="category"></span>
      <span id="score">النقاط: 0</span>
    </div>

    <div id="question" class="question"></div>
    <div id="options" class="options"></div>
    <div id="feedback" aria-live="polite"></div>
    <button id="next" disabled>السؤال التالي</button>
    <p class="note">لن يتكرر السؤال على هذا الجهاز قبل مرور 15 دقيقة.</p>
  </main>

  <script>
    const questions = [
      { category: "Subject pronouns", q: "Maria is a teacher. ___ is kind.", options: ["He", "She", "They", "We"], answer: "She" },
      { category: "Subject pronouns", q: "Tom and I are students. ___ are in class.", options: ["We", "They", "He", "I"], answer: "We" },
      { category: "Subject pronouns", q: "My parents are at home. ___ are cooking.", options: ["You", "He", "They", "It"], answer: "They" },
      { category: "Possessive adjectives", q: "I have a new phone. This is ___ phone.", options: ["my", "me", "I", "mine"], answer: "my" },
      { category: "Possessive adjectives", q: "Anna has a brother. ___ brother is funny.", options: ["Her", "His", "Their", "Our"], answer: "Her" },
      { category: "Possessive adjectives", q: "We live here. This is ___ house.", options: ["our", "us", "we", "ours"], answer: "our" },
      { category: "Verbs", q: "He ___ coffee every morning.", options: ["drink", "drinks", "drinking", "to drink"], answer: "drinks" },
      { category: "Verbs", q: "They ___ English at work.", options: ["study", "studies", "studying", "is study"], answer: "study" },
      { category: "Verbs", q: "I ___ in a small office.", options: ["work", "works", "working", "to works"], answer: "work" },
      { category: "Prepositions", q: "The keys are ___ the table.", options: ["on", "at", "to", "under"], answer: "on" },
      { category: "Prepositions", q: "The class starts ___ 9 o’clock.", options: ["in", "on", "at", "from"], answer: "at" },
      { category: "Prepositions", q: "She lives ___ London.", options: ["on", "in", "at", "to"], answer: "in" },
      { category: "Conjunctions", q: "I like tea ___ coffee.", options: ["and", "but", "because", "or"], answer: "and" },
      { category: "Conjunctions", q: "He is tired, ___ he is happy.", options: ["but", "and", "because", "or"], answer: "but" },
      { category: "Conjunctions", q: "We can have tea ___ coffee.", options: ["or", "because", "but", "so"], answer: "or" },
      { category: "Responses: yes / no", q: "Are you a student? ___, I am.", options: ["Yes", "No", "Other", "Too"], answer: "Yes" },
      { category: "Responses: yes / no", q: "Do you drink coffee? ___, I don’t.", options: ["Yes", "No", "Too", "Other"], answer: "No" },
      { category: "Not / too / other", q: "I ___ like cold tea.", options: ["not", "too", "other", "yes"], answer: "not" },
      { category: "Not / too / other", q: "I want a cup of tea, ___.", options: ["too", "not", "other", "no"], answer: "too" },
      { category: "Not / too / other", q: "One bag is black. The ___ bag is brown.", options: ["other", "too", "not", "yes"], answer: "other" },
      { category: "Adverbs", q: "She speaks English ___.", options: ["slowly", "slow", "slowness", "slower"], answer: "slowly" },
      { category: "Adverbs", q: "I ___ eat breakfast before work.", options: ["always", "early", "friendly", "blue"], answer: "always" },
      { category: "Appearance", q: "A person who is 190 cm tall is ___.", options: ["tall", "short", "old", "quiet"], answer: "tall" },
      { category: "Appearance", q: "Her hair has many curls. She has ___ hair.", options: ["curly", "straight", "tall", "friendly"], answer: "curly" },
      { category: "Personality", q: "A person who likes helping people is ___.", options: ["kind", "curly", "short", "young"], answer: "kind" },
      { category: "Personality", q: "A person who enjoys talking to people is ___.", options: ["friendly", "tall", "straight", "old"], answer: "friendly" }
    ];

    const STORAGE_KEY = "a1EnglishGameAskedQuestions";
    const WAIT_TIME = 15 * 60 * 1000;
    let currentQuestion = null;
    let score = 0;
    let answered = false;

    const questionEl = document.getElementById("question");
    const optionsEl = document.getElementById("options");
    const categoryEl = document.getElementById("category");
    const feedbackEl = document.getElementById("feedback");
    const scoreEl = document.getElementById("score");
    const nextBtn = document.getElementById("next");

    function getAskedQuestions() {
      try {
        return JSON.parse(localStorage.getItem(STORAGE_KEY)) || {};
      } catch {
        return {};
      }
    }

    function saveAskedQuestion(id) {
      const asked = getAskedQuestions();
      asked[id] = Date.now();
      localStorage.setItem(STORAGE_KEY, JSON.stringify(asked));
    }

    function loadQuestion() {
      answered = false;
      nextBtn.disabled = true;
      feedbackEl.textContent = "";
      optionsEl.innerHTML = "";

      const asked = getAskedQuestions();
      const now = Date.now();
      const available = questions
        .map((item, id) => ({ ...item, id }))
        .filter(item => !asked[item.id] || now - asked[item.id] >= WAIT_TIME);

      if (available.length === 0) {
        const earliest = Math.min(...Object.values(asked));
        const remaining = Math.max(0, WAIT_TIME - (now - earliest));
        const minutes = Math.ceil(remaining / 60000);
        questionEl.textContent = `انتهت الأسئلة المتاحة الآن. جرّب بعد ${minutes} دقيقة.`;
        categoryEl.textContent = "";
        return;
      }

      currentQuestion = available[Math.floor(Math.random() * available.length)];
      saveAskedQuestion(currentQuestion.id);

      categoryEl.textContent = currentQuestion.category;
      questionEl.textContent = currentQuestion.q;

      const shuffledOptions = [...currentQuestion.options]
        .sort(() => Math.random() - 0.5);

      shuffledOptions.forEach(option => {
        const button = document.createElement("button");
        button.className = "option";
        button.textContent = option;
        button.onclick = () => chooseAnswer(button, option);
        optionsEl.appendChild(button);
      });
    }

    function chooseAnswer(button, choice) {
      if (answered) return;
      answered = true;

      const buttons = [...optionsEl.querySelectorAll("button")];
      buttons.forEach(item => {
        item.disabled = true;
        if (item.textContent === currentQuestion.answer) item.classList.add("correct");
      });

      if (choice === currentQuestion.answer) {
        feedbackEl.textContent = "صحيح! أحسنت 🎉";
        feedbackEl.style.color = "#15803d";
        score++;
        scoreEl.textContent = `النقاط: ${score}`;
      } else {
        button.classList.add("wrong");
        feedbackEl.textContent = `إجابة خطأ. الإجابة الصحيحة: ${currentQuestion.answer}`;
        feedbackEl.style.color = "#b91c1c";
      }

      nextBtn.disabled = false;
    }

    nextBtn.addEventListener("click", loadQuestion);
    loadQuestion();
  </script>
</body>
</html>
