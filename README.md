# -
Az
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8" />
<title>محادثة سامر و أمنيه</title>
<style>
    body {
        background: #1c0f27;
        font-family: sans-serif;
        color: #fff;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
        margin: 0;
    }

    .chat-box {
        width: 95%;
        max-width: 450px;
        background: rgba(40,20,60,0.8);
        padding: 20px;
        border-radius: 20px;
        height: 85vh;
        overflow-y: auto;
    }

    .msg {display:flex; margin-bottom:15px; max-width:85%; animation:fade .3s;}
    @keyframes fade {from{opacity:0; transform:translateY(10px);}to{opacity:1; transform:translateY(0);}}

    .bot p, .user p {
        padding: 12px 18px;
        border-radius: 20px;
        font-size: 1.05em;
        box-shadow: 0 3px 6px rgba(0,0,0,0.3);
    }

    .bot p {background:#6a1a7f; color:#fff; border-top-left-radius:5px;}
    .user {justify-content:flex-end; text-align:right;}
    .user p {background:#ff69b4; color:#000; border-bottom-right-radius:5px;}

    .avatar {
        width: 45px;
        height: 45px;
        border-radius: 50%;
        background: #ff69b4;
        display:flex;
        justify-content:center;
        align-items:center;
        font-size:22px;
        margin-left:10px;
        flex-shrink:0;
    }

    .options {margin-top:20px; text-align:center;}
    .options button {
        background:#ff69b4;
        border:none;
        padding:12px 25px;
        margin:5px;
        border-radius:25px;
        font-weight:bold;
        cursor:pointer;
    }
    
    /* 🔴 تصميم مؤشر الكتابة (Typing Indicator) */
    .typing-indicator {
        display: flex;
        align-items: center;
        max-width: 85%;
        margin-bottom: 15px;
        /* استخدام نفس أسلوب .msg */
        animation: fade 0.3s;
    }
    
    /* تنسيق صندوق المؤشر ليكون مثل فقاعة الرسالة */
    .typing-indicator .indicator-box {
        background: #6a1a7f;
        padding: 12px 18px;
        border-radius: 20px;
        border-top-left-radius: 5px;
        box-shadow: 0 3px 6px rgba(0,0,0,0.3);
        display: flex;
        align-items: center;
        height: 10px; /* لتقليل ارتفاع الفقاعة */
    }

    .typing-indicator .dot {
        width: 8px;
        height: 8px;
        background: #fff; /* لون النقاط أبيض */
        border-radius: 50%;
        margin: 0 2px;
        opacity: 0.2; /* تبدأ بنسبة شفافة */
        animation: blink 1s infinite;
    }
    
    .typing-indicator .dot:nth-child(2) { animation-delay: 0.2s; }
    .typing-indicator .dot:nth-child(3) { animation-delay: 0.4s; }

    /* حركة وميض النقاط */
    @keyframes blink {
        0%, 100% { opacity: 0.2; }
        50% { opacity: 1; }
    }
</style>
</head>
<body>

<div class="chat-box" id="chat"></div>

<script>
const chat = document.getElementById("chat");

// دالة إضافة رسالة عادية
function addMsg(text, sender){
    const box = document.createElement("div");
    box.className = "msg " + sender;

    if(sender === "bot"){
        const av = document.createElement("div");
        av.className = "avatar";
        av.textContent = "🎀"; // فيونكة وردية لافتار أمنيه
        box.appendChild(av);
    }

    const p = document.createElement("p");
    p.textContent = text;
    box.appendChild(p);
    chat.appendChild(box);
    chat.scrollTop = chat.scrollHeight;
}

// دالة إضافة الخيارات
function addOptions(opts){
    const op = document.createElement("div");
    op.className = "options";
    opts.forEach(o=>{
        const b = document.createElement("button");
        b.textContent = o.t;
        // عند النقر على خيار، نزيل الخيارات ونتابع للخطوة التالية
        b.onclick = ()=>{ op.remove(); userChoice(o.t, o.next); };
        op.appendChild(b);
    });
    chat.appendChild(op);
    chat.scrollTop = chat.scrollHeight;
}

// دالة إظهار مؤشر الكتابة
function addTypingIndicator() {
    const typingBox = document.createElement("div");
    typingBox.id = "typing-msg"; // لتسهيل إزالته لاحقاً
    typingBox.className = "typing-indicator";

    // Avatar
    const av = document.createElement("div");
    av.className = "avatar";
    av.textContent = "🎀";
    typingBox.appendChild(av);

    // Indicator Box (الفقاعة التي تحتوي على النقاط)
    const indicatorBox = document.createElement("div");
    indicatorBox.className = "indicator-box";
    
    // النقاط المتحركة
    for(let i = 0; i < 3; i++) {
        const dot = document.createElement("span");
        dot.className = "dot";
        indicatorBox.appendChild(dot);
    }
    
    typingBox.appendChild(indicatorBox);
    chat.appendChild(typingBox);
    chat.scrollTop = chat.scrollHeight;
}

// دالة إزالة مؤشر الكتابة
function removeTypingIndicator() {
    const typingBox = document.getElementById("typing-msg");
    if (typingBox) {
        typingBox.remove();
    }
}

// دالة اختيار المستخدم (تضيف رسالة المستخدم ثم تنتقل للخطوة التالية)
function userChoice(text, go){
    addMsg(text, "user");
    // يتم التأخير هنا قبل الانتقال للخطوة التالية لإظهار الكتابة
    setTimeout(()=> step(go), 500); // 500ms لتأخير بسيط بعد رسالة المستخدم
}

// دالة الانتقال للخطوة التالية (تم تعديلها لإضافة مؤشر الكتابة)
function step(k){
    const s = S[k];
    
    // 1. إظهار مؤشر الكتابة مباشرة
    addTypingIndicator();

    // 2. الانتظار لمدة 5 ثواني ثم عرض الرسالة والخيارات
    const typingDuration = 5000; // 5000 مللي ثانية = 5 ثواني
    
    setTimeout(() => {
        removeTypingIndicator(); // إزالة المؤشر

        // عرض الرسالة والخيارات الفعلية
        addMsg(s.m, "bot");
        if(s.o && s.o.length > 0) addOptions(s.o);
    }, typingDuration); 
}

// ---------- السيناريو (كما هو لم يتغير) ----------
const S = {
 start: {
   m:"هلو أنتِ أمنيه",
   o:[{t:"اي اني امنيه 🎀", next:"yes_om"},{t:"لا اني مو امنيه", next:"no_om"}]
 },

 no_om:{
   m:"لا أنتِ أمنيه اعرفج يحلوة 🫦",
   o:[{t:"تمام شنو سؤالك؟", next:"ask"}]
 },

 yes_om:{
   m:"عندي سؤال مهم",
   o:[{t:"شنو هو", next:"what"},{t:"ما اريد اعرف", next:"force"}]
 },

 ask:{
   m:"عندي سؤال مهم",
   o:[{t:"شنو هو", next:"what"},{t:"ما اريد اعرف", next:"force"}]
 },

 force:{
   m:"غصباً عليج تعرفين 🌚",
   o:[{t:"تمام شنو هو؟", next:"love_q"}]
 },

 what:{ m:"تحبين سامر؟", o:[
   {t:"اي احبه", next:"like"},
   {t:"اموت عليه", next:"like"},
   {t:"أعشقه", next:"like"},
   {t:"لا ما احبه", next:"no_like"}
 ]},

 love_q:{ m:"تحبين سامر؟", o:[
   {t:"اي احبه", next:"like"},
   {t:"اموت عليه", next:"like"},
   {t:"أعشقه", next:"like"},
   {t:"لا ما احبه", next:"no_like"}
 ]},

 like:{
   m:"ادري بيج.. أصلاً هو هم يحبج ❤️",
   o:[{t:"مااا اني أكثر", next:"more1"}]
 },

 no_like:{
   m:"شنو ماتحبينه بكيفج قابل؟",
   o:[{t:"اي بكيفي", next:"forced_love"}]
 },

 forced_love:{
   m:"لا تحبينه غصباً عليج",
   o:[{t:"اي احبه", next:"like"}]
 },

 more1:{
   m:"لاا هو أكثر",
   o:[{t:"لاا أنيي أكثرر", next:"more2"}]
 },

 more2:{
   m:"يلا تمام 🌚😂",
   o:[]
 }
};

// بدء المحادثة
step("start");
</script>
</body>
</html>
