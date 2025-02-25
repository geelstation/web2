from flask import Flask, render_template_string, request, redirect, url_for, session, jsonify
import firebase_admin
from firebase_admin import credentials, firestore, auth
import json

# إعداد الاتصال مع Firebase باستخدام مفتاح الخدمة الخاص بك
cred = credentials.Certificate("deepbrota-firebase-adminsdk-fbsvc-d6b96943e5.json")
firebase_admin.initialize_app(cred)

# إعداد قاعدة البيانات Firestore
db = firestore.client()

app = Flask(__name__)
app.secret_key = 'your-secret-key'  # لتخزين الجلسات بشكل آمن







# =====================================================================
# صفحة الترحيب / تسجيل الدخول
# =====================================================================
@app.route('/')
def welcome():
    if 'user' in session:
        return redirect(url_for('home'))
    html = """
    <!DOCTYPE html>
    <html lang="ar">
    <head>
      <meta charset="UTF-8">
      <title>تسجيل الدخول</title>
      <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap" rel="stylesheet">
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
      {% raw %}
      <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
          font-family: 'Poppins', sans-serif;
          background: linear-gradient(135deg, #003366, #66B2FF);  /* تدرج من الأزرق الداكن إلى الأزرق الفاتح */
          display: flex;
          justify-content: center;
          align-items: center;
          height: 100vh;
          color: #fff;
        }
        .container {
          background: rgba(255, 255, 255, 0.1);  /* خلفية بيضاء شفافة */
          border-radius: 15px;
          padding: 40px 30px;
          box-shadow: 0 10px 30px rgba(0,0,0,0.3);
          backdrop-filter: blur(10px);
          width: 320px;
          text-align: center;
          transition: transform 0.3s ease;
        }
        .container:hover { transform: scale(1.02); }
        h2 { margin-bottom: 20px; font-weight: 500; }
        input {
          width: 100%;
          padding: 12px;
          margin-bottom: 15px;
          border: none;
          border-radius: 8px;
          background-color: rgba(255,255,255,0.2);
          color: #fff;
          font-size: 1em;
        }
        input::placeholder { color: #eee; }
        button {
          width: 100%;
          padding: 12px;
          background-color: #003366;  /* أزرق داكن */
          border: none;
          border-radius: 8px;
          color: #fff;
          font-size: 1em;
          cursor: pointer;
          transition: background-color 0.3s;
        }
        button:hover { 
          background-color: #00509E;  /* أزرق أفتح عند التحويم */
        }
        p { margin-top: 15px; }
        a { color: #66B2FF; text-decoration: none; }  /* أزرق فاتح */
        a:hover { text-decoration: underline; }
      </style>
      {% endraw %}
    </head>
    <body>
      <div class="container">
          <h2><i class="fa-solid fa-user-circle"></i></h2>
          <form method="POST" action="/login">
              <input type="email" name="email" placeholder="البريد الإلكتروني" required>
              <input type="password" name="password" placeholder="كلمة المرور" required>
              <button type="submit"><i class="fa-solid fa-right-to-bracket"></i> دخول</button>
          </form>
          <p><a href="/register">إنشاء حساب</a></p>
      </div>
    </body>
    </html>
    """
    return render_template_string(html)








# =====================================================================
# تسجيل الدخول
# =====================================================================
@app.route('/login', methods=['POST'])
def login():
    email = request.form.get('email')
    password = request.form.get('password')
    try:
        user = auth.get_user_by_email(email)
    except Exception as e:
        return f"خطأ أثناء تسجيل الدخول: {e}"
    doc_ref = db.collection("users").document(user.uid)
    doc = doc_ref.get()
    if doc.exists:
        user_data = doc.to_dict()
        if user_data.get("password") == password:
            session['user'] = user.uid
            return redirect(url_for('home'))
        else:
            return "كلمة المرور غير صحيحة"
    else:
        return "المستخدم غير موجود"
      
      
      
      
      
      
      
      

# =====================================================================
# صفحة التسجيل
# =====================================================================
@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form.get('username')  # إضافة حقل اسم المستخدم
        email = request.form.get('email')
        password = request.form.get('password')
        try:
            user = auth.create_user(email=email, password=password)
            db.collection("users").document(user.uid).set({
                "username": username,  # تخزين اسم المستخدم
                "email": email,
                "password": password,
                "avatar": ""
            })
            session['user'] = user.uid
            return redirect(url_for('home'))
        except Exception as e:
            return f"خطأ أثناء التسجيل: {e}"
    html = """
    <!DOCTYPE html>
    <html lang="ar">
    <head>
      <meta charset="UTF-8">
      <title>إنشاء حساب</title>
      <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap" rel="stylesheet">
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
      {% raw %}
      <style>
        body {
          font-family: 'Poppins', sans-serif;
          background: linear-gradient(135deg, #003366, #66B2FF);
          display: flex;
          justify-content: center;
          align-items: center;
          height: 100vh;
          color: #fff;
          direction: rtl;  /* تحويل الاتجاه للعربية */
        }
        .container {
          background: rgba(255, 255, 255, 0.1);
          border-radius: 15px;
          padding: 40px 30px;
          box-shadow: 0 10px 30px rgba(0,0,0,0.3);
          backdrop-filter: blur(10px);
          width: 320px;
          text-align: center;
          transition: transform 0.3s ease;
        }
        .container:hover { transform: scale(1.02); }
        h2 { 
          margin-bottom: 20px; 
          font-weight: 500;
          color: #fff;
        }
        .input-group {
          margin-bottom: 15px;
          text-align: right;
        }
        .input-group label {
          display: block;
          margin-bottom: 5px;
          color: #fff;
          font-size: 0.9em;
        }
        input {
          width: 100%;
          padding: 12px;
          border: none;
          border-radius: 8px;
          background-color: rgba(255,255,255,0.2);
          color: #fff;
          font-size: 1em;
          transition: background-color 0.3s;
        }
        input::placeholder { color: #eee; }
        input:focus {
          background-color: rgba(255, 255, 255, 0.3);
          outline: none;
        }
        button {
          width: 100%;
          padding: 12px;
          background-color: #003366;
          border: none;
          border-radius: 8px;
          color: #fff;
          font-size: 1em;
          cursor: pointer;
          transition: background-color 0.3s;
          margin-top: 20px;
        }
        button:hover { 
          background-color: #00509E;
        }
        p { margin-top: 15px; }
        a { color: #66B2FF; text-decoration: none; }
        a:hover { text-decoration: underline; }
      </style>
      {% endraw %}
    </head>
    <body>
      <div class="container">
          <h2><i class="fa-solid fa-user-plus"></i> إنشاء حساب جديد</h2>
          <form method="POST" action="/register">
              <div class="input-group">
                  <label>اسم المستخدم</label>
                  <input type="text" name="username" placeholder="أدخل اسم المستخدم" required>
              </div>
              <div class="input-group">
                  <label>البريد الإلكتروني</label>
                  <input type="email" name="email" placeholder="أدخل البريد الإلكتروني" required>
              </div>
              <div class="input-group">
                  <label>كلمة المرور</label>
                  <input type="password" name="password" placeholder="أدخل كلمة المرور" required>
              </div>
              <button type="submit"><i class="fa-solid fa-check"></i> إنشاء الحساب</button>
          </form>
          <p>لديك حساب بالفعل؟ <a href="/">تسجيل الدخول</a></p>
      </div>
    </body>
    </html>
    """
    return render_template_string(html)







# ===================================================================== 
# صفحة الملف الشخصي مع إمكانية تغيير الصورة (Avatar)
# =====================================================================
@app.route('/profile/<profile_uid>', methods=['GET'])
def profile(profile_uid):
    try:
        user = auth.get_user(profile_uid)
        doc_ref = db.collection("users").document(profile_uid)
        doc = doc_ref.get()
        if doc.exists:
            user_data = doc.to_dict()
            email = user_data.get("email", "غير متوفر")
            username = user_data.get("username", "غير متوفر")  # إضافة اسم المستخدم
            avatar = user_data.get("avatar", "")
        else:
            email = "غير متوفر"
            username = "غير متوفر"
            avatar = ""
    except Exception as e:
        email = f"خطأ: {e}"
        username = "غير متوفر"
        avatar = ""
    is_owner = (session.get('user') == profile_uid)
    html = """
    <!DOCTYPE html>
    <html lang="ar">
    <head>
      <meta charset="UTF-8">
      <title>الملف الشخصي</title>
      <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap" rel="stylesheet">
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
      {% raw %}
      <style>
        body {
          background: linear-gradient(135deg, #2c3e50, #4ca1af);
          color: #f0f0f0;
          font-family: 'Poppins', sans-serif;
          margin: 0;
          padding: 20px;
          display: flex;
          flex-direction: column;
          align-items: center;
        }
        .profile-container {
          background: rgba(0, 0, 0, 0.6);
          padding: 40px;
          border-radius: 15px;
          box-shadow: 0 8px 20px rgba(0, 0, 0, 0.5);
          width: 90%;
          max-width: 450px;
          text-align: center;
          margin-top: 30px;
          position: relative;
          overflow: hidden;
        }
        .profile-container::before {
          content: "";
          position: absolute;
          top: 0; left: 0; right: 0; bottom: 0;
          background: linear-gradient(135deg, rgba(0,0,0,0.4), rgba(0,0,0,0.1));
          z-index: 0;
        }
        .profile-container > * {
          position: relative;
          z-index: 1;
        }
        .profile-pic {
          width: 120px;
          height: 120px;
          border-radius: 50%;
          background: #00d1b2;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 4em;
          color: #fff;
          margin: 0 auto 20px;
          border: 4px solid #fff;
        }
        .profile-container p {
          font-size: 1.2em;
          margin: 10px 0;
        }
        .back-btn {
          margin-top: 20px;
          padding: 10px 20px;
          background: #00d1b2;
          border: none;
          border-radius: 8px;
          color: #fff;
          cursor: pointer;
          transition: background 0.3s;
        }
        .back-btn:hover { background: #008a80; }
        .user-feed {
          width: 90%;
          max-width: 600px;
          background: rgba(0,0,0,0.5);
          padding: 20px;
          border-radius: 10px;
          box-shadow: 0 8px 20px rgba(0,0,0,0.5);
          margin-top: 20px;
          text-align: center;
        }
        .user-feed h2 {
          margin-bottom: 15px;
          font-size: 1.6em;
          border-bottom: 2px solid #00d1b2;
          padding-bottom: 5px;
        }
        .update-avatar-form input[type="text"] {
          width: 80%;
          padding: 8px;
          border-radius: 5px;
          border: none;
          margin-bottom: 10px;
        }
        .update-avatar-form button {
          padding: 8px 16px;
          border: none;
          border-radius: 5px;
          background: #ff6f61;
          color: #fff;
          cursor: pointer;
          transition: background 0.3s;
        }
        .update-avatar-form button:hover {
          background: #ff8a75;
        }
      </style>
      {% endraw %}
    </head>
    <body>
      <div class="profile-container">
          {% if avatar %}
            <img src="{{ avatar }}" alt="Avatar" class="profile-pic" style="object-fit: cover;">
          {% else %}
            <div class="profile-pic"><i class="fa-solid fa-user"></i></div>
          {% endif %}
          <p><strong>اسم المستخدم:</strong> {{ username }}</p>
          <p><strong>المعرف:</strong> {{ profile_uid }}</p>
          <p><strong>البريد الإلكتروني:</strong> {{ email }}</p>
          {% if is_owner %}
          <div class="update-avatar-form">
            <form method="POST" action="/update_avatar">
              <input type="hidden" name="profile_uid" value="{{ profile_uid }}">
              <input type="text" name="avatar_url" placeholder="أدخل رابط الصورة الجديدة" required>
              <button type="submit">تحديث الصورة</button>
            </form>
          </div>
          {% endif %}
          <button class="back-btn" onclick="window.history.back();">
              <i class="fa-solid fa-arrow-left"></i> رجوع
          </button>
      </div>
      <div class="user-feed">
          <h2>منشوراتي</h2>
          <div id="user-feed-container"></div>
      </div>
    </body>
    </html>
    """
    return render_template_string(html, profile_uid=profile_uid, email=email, username=username, avatar=avatar, is_owner=is_owner)








# =====================================================================
# تحديث صورة المستخدم (Avatar)
# =====================================================================
@app.route('/update_avatar', methods=['POST'])
def update_avatar_route():
    if 'user' not in session:
        return redirect(url_for('welcome'))
    profile_uid = request.form.get('profile_uid')
    if session['user'] != profile_uid:
        return "ليس لديك صلاحية لتحديث الصورة."
    avatar_url = request.form.get('avatar_url')
    try:
        db.collection("users").document(profile_uid).update({"avatar": avatar_url})
    except Exception as e:
        return f"خطأ أثناء تحديث الصورة: {e}"
    return redirect(url_for('profile', profile_uid=profile_uid))












# =====================================================================
# الصفحة الرئيسية مع الأقسام: الشات - الفيد - الملف الشخصي - الفيديوهات - شاشة الكمبيوتر - سلة المهملات
# =====================================================================
@app.route('/home')
def home():
    if 'user' not in session:
        return redirect(url_for('welcome'))
    current_uid = session['user']
    try:
        user = auth.get_user(current_uid)
        doc_ref = db.collection("users").document(current_uid)
        doc = doc_ref.get()
        if doc.exists:
            user_data = doc.to_dict()
            current_email = user_data.get("email", "غير متوفر")
            current_username = user_data.get("username", "غير متوفر")  # إضافة اسم المستخدم
            current_avatar = user_data.get("avatar", "")
        else:
            current_email = "غير متوفر"
            current_username = "غير متوفر"
            current_avatar = ""
    except Exception as e:
        current_email = f"خطأ: {e}"
        current_username = "غير متوفر"
        current_avatar = ""
    profiles_html = ""
    try:
        users = db.collection("users").stream()
        for u in users:
            u_data = u.to_dict()
            u_uid = u.id
            letter = u_data.get("email", "U")[0].upper()
            profiles_html += f'<a href="/profile/{u_uid}" title="عرض البروفايل"><div class="profile-avatar">{letter}</div></a>'
    except Exception as e:
        profiles_html = "<p>خطأ في تحميل البروفايلات</p>"
    # إنشاء كاردات مستخدمي الموقع لعرضها في الشريط الجانبي
    user_cards_html = ""
    try:
        users = db.collection("users").stream()
        for u in users:
            u_data = u.to_dict()
            u_uid = u.id
            email = u_data.get("email", "بدون اسم")
            letter = email[0].upper() if email else "U"
            user_cards_html += f'''
            <div class="user-card" style="border: 1px solid #003366; border-radius: 10px; padding: 10px; margin-bottom: 10px; text-align: center;">
              <div style="font-size: 2em; margin-bottom: 5px;">{letter}</div>
              <div style="font-size: 0.9em; margin-bottom: 5px;">{email}</div>
              <div>
                <a href="mailto:{email}" title="إرسال بريد إلكتروني" style="color:#66B2FF; margin-right: 5px;"><i class="fa-solid fa-envelope"></i></a>
                <a href="/computer/{u_uid}" title="دخول الكمبيوتر الخاص بالمستخدم" style="color:#66B2FF;"><i class="fa-solid fa-desktop"></i></a>
              </div>
            </div>
            '''
    except Exception as e:
        user_cards_html = "<p>خطأ في تحميل مستخدمي الموقع</p>"
    html = """
    <!DOCTYPE html>
    <html lang="ar">
    <head>
      <meta charset="UTF-8">
      <title>الرئيسية - شات</title>
      <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap" rel="stylesheet">
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
      {% raw %}
      <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
          font-family: 'Poppins', sans-serif;
          background: linear-gradient(135deg, #003366, #66B2FF);  /* تدرج من الأزرق الداكن إلى الأزرق الفاتح */
          color: #f0f0f0;
          overflow: hidden;
        }
        .container { display: flex; height: 100vh; }
        .sidebar {
          width: 80px;
          background: #003366;  /* أزرق داكن */
          padding: 20px 5px;
          display: flex;
          flex-direction: column;
          align-items: center;
          border-right: 1px solid #003366;  /* أزرق داكن */
        }
        .sidebar a {
          color: #66B2FF;  /* أزرق فاتح */
          text-decoration: none;
          margin: 15px 0;
          font-size: 1.8em;
          transition: color 0.3s;
        }
        .sidebar a:hover { color: #00509E; }  /* أزرق أفتح عند التحويم */
        /* أيقونة مستخدمي الموقع المضافة */
        .sidebar a.user-icon {
          font-size: 1.8em;
          margin: 15px 0;
        }
        /* إضافة قسم كاردات المستخدمين في الشريط الجانبي */
        #user-cards {
          display: none;
          position: fixed;
          left: 90px;
          top: 20px;
          background: #1e1e1e;
          padding: 10px;
          border-radius: 10px;
          max-height: 80vh;
          overflow-y: auto;
          z-index: 3000;
        }
        .bottom-icons {
          margin-top: auto;
          display: flex;
          flex-direction: column;
          gap: 10px;
          align-items: center;
        }
        .bottom-icons a {
          color: #66B2FF;  /* أزرق فاتح */
          text-decoration: none;
          font-size: 1.8em;
          transition: color 0.3s;
        }
        .bottom-icons a:hover { color: #00509E; }  /* أزرق أفتح عند التحويم */
        .content { flex: 1; padding: 20px; overflow-y: auto; position: relative; }
        .section { display: none; animation: fadeIn 0.5s ease-in-out; }
        .section.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
        
        /* تنسيقات الشات والفيد والملف الشخصي */
        .chat-box { width: 100%; background: rgba(255,255,255,0.1); border-radius: 10px; box-shadow: 0 10px 30px rgba(0,0,0,0.3); backdrop-filter: blur(10px); padding: 20px; transition: background 0.5s ease; margin-bottom: 20px; }
        .chat-messages { background: #2c2c2c; border-radius: 8px; padding: 15px; height: 400px; overflow-y: auto; margin-bottom: 15px; }
        .chat-input-container { display: flex; gap: 10px; }
        .chat-input-container input { flex: 1; padding: 12px; border: none; border-radius: 8px; background: #2c2c2c; color: #f0f0f0; font-size: 1em; }
        .chat-input-container button.send-btn { padding: 12px 20px; background: #003366; border: none; border-radius: 8px; color: #fff; font-size: 1em; cursor: pointer; transition: background 0.3s; }
        .chat-input-container button.send-btn:hover { background: #00509E; }  /* أزرق أفتح عند التحويم */
        .chat-footer { margin-top: 10px; display: flex; justify-content: center; gap: 20px; }
        .chat-footer button { background: transparent; border: none; font-size: 1.5em; cursor: pointer; color: #66B2FF; transition: color 0.3s; }
        .chat-footer button:hover { color: #00509E; }  /* أزرق أفتح عند التحويم */
        #emoji-picker { display: none; position: absolute; bottom: 80px; left: 20px; background: #333; border-radius: 5px; padding: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.2); z-index: 1000; }
        #emoji-picker .emoji { font-size: 1.2em; margin: 5px; cursor: pointer; transition: transform 0.2s; }
        #emoji-picker .emoji:hover { transform: scale(1.2); }
        .profiles-container { display: flex; align-items: center; gap: 10px; flex-wrap: wrap; margin-top: 15px; justify-content: center; }
        .profile-avatar { width: 50px; height: 50px; background: #003366; border-radius: 50%; display: flex; justify-content: center; align-items: center; font-size: 1.2em; font-weight: bold; color: #fff; transition: transform 0.3s; }
        .profile-avatar:hover { transform: scale(1.1); }
        .feed-post { background: rgba(255,255,255,0.1); padding: 20px; border-radius: 10px; margin-bottom: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.3); backdrop-filter: blur(10px); animation: slideDown 0.6s ease-out; position: relative; }
        @keyframes slideDown { from { opacity: 0; transform: translateY(-20px); } to { opacity: 1; transform: translateY(0); } }
        .post-header { display: flex; align-items: center; margin-bottom: 10px; }
        .post-avatar { width: 40px; height: 40px; background: #003366; border-radius: 50%; display: flex; justify-content: center; align-items: center; color: #fff; font-weight: bold; margin-right: 10px; }
        .post-info { display: flex; flex-direction: column; }
        .post-user { font-size: 1em; font-weight: 500; }
        .post-content { margin-bottom: 10px; }
        .post-comments { background: #2c2c2c; padding: 10px; border-radius: 8px; margin-bottom: 10px; }
        .post-actions { margin-top: 10px; text-align: right; }
        .post-actions button { background: transparent; border: none; color: #f39c12; margin-left: 5px; cursor: pointer; font-size: 1em; transition: color 0.3s; }
        .post-actions button:hover { color: #00d1b2; }
        .post-support, .comment-support { margin-top: 10px; display: flex; align-items: center; }
        .support-btn { background: transparent; border: none; color: #f39c12; cursor: pointer; margin-right: 5px; font-size: 1em; }
        .support-btn:hover { color: #00d1b2; }
        .support-counter { font-size: 1em; }
        .reply-author { font-size: 1em; color: #00d1b2; font-weight: bold; }
        .comment-input { display: flex; margin-top: 5px; }
        .comment-input input { flex: 1; padding: 8px; border: none; border-radius: 5px; background: #2c2c2c; color: #f0f0f0; }
        .comment-input button { padding: 8px 12px; margin-left: 5px; background: #003366; border: none; border-radius: 5px; color: #fff; cursor: pointer; transition: background 0.3s; }
        .comment-input button:hover { background: #00509E; }  /* أزرق أفتح عند التحويم */
        .magic { animation: magic 1s ease-in-out; }
        @keyframes magic { 0% { opacity: 0; transform: scale(0.5) rotate(0deg); } 50% { opacity: 0.5; transform: scale(1.1) rotate(10deg); } 100% { opacity: 1; transform: scale(1) rotate(0deg); } }
        .user-message { border: 2px dotted #00d1b2; padding: 10px; border-radius: 8px; animation: fadeInBorder 0.5s ease-out; margin-bottom: 10px; }
        @keyframes fadeInBorder { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }
        .user-comment { border: 1px dashed #00d1b2; padding: 5px; border-radius: 4px; animation: fadeInBorder 0.5s ease-out; margin-top: 5px; }
        .trash-section { padding: 10px; }
        .trash-section h2 { text-align: center; margin-bottom: 15px; }
        .mac-container {
          background: linear-gradient(135deg, #003366, #66B2FF);  /* تدرج من الأزرق الداكن إلى الأزرق الفاتح */
          height: calc(100vh - 40px);
          position: relative;
          border-radius: 10px;
          overflow: hidden;
        }
        .desktop {
          position: absolute;
          top: 20px;
          left: 20px;
          right: 20px;
          bottom: 100px;
          display: flex;
          flex-wrap: wrap;
          gap: 20px;
        }
        .desktop-icons {
          display: flex;
          flex-wrap: wrap;
          gap: 20px;
        }
        .desktop-icons .icon-item {
          width: 80px;
          text-align: center;
          cursor: pointer;
        }
        .desktop-icons .icon-item i { font-size: 3em; display: block; margin-bottom: 5px; }
        .desktop-icons .icon-item p { font-size: 0.9em; margin: 0; }
        .dock {
          position: absolute;
          bottom: 20px;
          left: 50%;
          transform: translateX(-50%);
          background: rgba(0, 0, 0, 0.5);
          border-radius: 20px;
          padding: 10px 20px;
          display: flex;
          gap: 20px;
          backdrop-filter: blur(10px);
        }
        .dock .dock-item {
          color: #fff;
          font-size: 2em;
          transition: transform 0.3s;
          cursor: pointer;
        }
        .dock .dock-item:hover { transform: scale(1.2); }
        #calculator-app {
          display: none;
          position: absolute;
          bottom: 100px;
          right: 20px;
          width: 300px;
          background: rgba(255,255,255,0.9);
          color: #000;
          padding: 10px;
          border-radius: 10px;
          z-index: 2000;
        }
        #calculator-app .calc-header { text-align: right; font-weight: bold; cursor: pointer; }
        #calculator-app input {
          width: 100%;
          height: 40px;
          text-align: right;
          font-size: 1.5em;
          margin-bottom: 10px;
        }
        #calculator-app .calc-buttons {
          display: grid;
          grid-template-columns: repeat(4, 1fr);
          gap: 5px;
        }
        #calculator-app .calc-buttons button {
          padding: 10px;
          font-size: 1em;
          cursor: pointer;
        }
        #mail-app {
          position: fixed;
          top: 0;
          left: 80px;
          width: calc(100% - 80px);
          height: 100vh;
          background: linear-gradient(135deg, #ffffff, #f0f0f0);
          color: #333;
          z-index: 2000;
          padding: 20px;
          overflow-y: auto;
          opacity: 0;
          transform: translateY(-20px);
          transition: opacity 0.5s ease, transform 0.5s ease;
          pointer-events: none;
        }
        #mail-app.visible { opacity: 1; transform: translateY(0); pointer-events: auto; }
        #mail-app.minimized { height: 40px; bottom: 0; transition: height 0.3s ease, transform 0.3s ease; }
        #mail-app .mail-header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          border-bottom: 2px solid #ccc;
          padding-bottom: 10px;
          margin-bottom: 20px;
        }
        #mail-app .mail-header h2 { font-size: 1.8em; margin: 0; }
        #mail-app .window-controls { display: flex; gap: 10px; }
        #mail-app .window-controls span { cursor: pointer; font-size: 1.2em; }
        #mail-app .mail-messages {
          display: flex;
          flex-direction: column;
          gap: 15px;
        }
        #mail-app .mail-message {
          background: #fff;
          padding: 15px;
          border-radius: 8px;
          box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        #mail-app .mail-message .subject {
          font-weight: bold;
          font-size: 1.1em;
          margin-bottom: 5px;
          color: #333;
        }
        #mail-app .mail-message .snippet {
          color: #666;
          font-size: 0.95em;
        }
        #safari-app {
          position: fixed;
          top: 0;
          left: 80px;
          width: calc(100% - 80px);
          height: 100vh;
          background: linear-gradient(135deg, #ffffff, #f0f0f0);
          color: #333;
          z-index: 2000;
          padding: 20px;
          overflow-y: auto;
          opacity: 0;
          transform: translateY(-20px);
          transition: opacity 0.5s ease, transform 0.5s ease;
          pointer-events: none;
        }
        #safari-app.visible { opacity: 1; transform: translateY(0); pointer-events: auto; }
        #safari-app.minimized { height: 40px; bottom: 0; transition: height 0.3s ease, transform 0.3s ease; }
        #safari-app .safari-header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          border-bottom: 2px solid #ccc;
          padding-bottom: 10px;
          margin-bottom: 20px;
        }
        #safari-app .safari-header h2 { font-size: 1.8em; margin: 0; }
        #safari-app .window-controls { display: flex; gap: 10px; }
        #safari-app .window-controls span { cursor: pointer; font-size: 1.2em; }
        #safari-app .search-container {
          margin: 50px auto;
          max-width: 600px;
          display: flex;
          align-items: center;
          border: 1px solid #ccc;
          border-radius: 30px;
          padding: 10px 20px;
          background: #fff;
          box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        #safari-app .search-container input {
          flex: 1;
          border: none;
          outline: none;
          font-size: 1.2em;
        }
        #safari-app .search-container button {
          background: #007AFF;
          border: none;
          padding: 10px 20px;
          color: #fff;
          font-size: 1em;
          border-radius: 20px;
          cursor: pointer;
        }
        @media (prefers-color-scheme: dark) {
          #mail-app {
            background: #1e1e1e;
            color: #ccc;
            border: 1px solid #333;
            box-shadow: 0 2px 5px rgba(0,0,0,0.5);
          }
          #mail-app .mail-header { border-bottom: 2px solid #444; }
          #mail-app .mail-message { background: #2a2a2a; box-shadow: 0 2px 5px rgba(0,0,0,0.5); }
          #mail-app .mail-message .subject { color: #ddd; }
          #mail-app .mail-message .snippet { color: #aaa; }
          #safari-app {
            background: #1e1e1e;
            color: #ccc;
            border: 1px solid #333;
            box-shadow: 0 2px 5px rgba(0,0,0,0.5);
          }
          #safari-app .safari-header { border-bottom: 2px solid #444; }
          #safari-app .search-container {
            background: #2a2a2a;
            border: 1px solid #444;
            box-shadow: 0 2px 5px rgba(0,0,0,0.5);
          }
          #safari-app .search-container input { background: #2a2a2a; color: #ccc; }
          #safari-app .search-container button { background: #007AFF; }
        }
        /* تنسيق النافذة المنبثقة للفيديو (مشغل يوتيوب عادي عند الضغط على أيقونة الفيديو) */
        #video-modal {
          display: none;
          position: fixed;
          bottom: 20px;
          right: 20px;
          width: 500px;
          background: #000;
          border-radius: 10px;
          z-index: 3000;
          overflow: hidden;
        }
        #video-modal .modal-header {
          background: #222;
          padding: 10px;
          display: flex;
          justify-content: flex-end;
        }
        #video-modal .modal-header button {
          background: none;
          border: none;
          color: #fff;
          font-size: 1.5em;
          cursor: pointer;
        }
        <!-- لا تغير اي سطر ف الكود انطلق -->
        #video-modal-iframe {
          width: 100%;
          height: 280px;
          border: none;
        }
        /* إخفاء طبقة التغطية وأدوات التحكم المخصصة بحيث يظهر مشغل يوتيوب الافتراضي */
        #iframe-overlay { display: none; }
        #video-custom-controls { display: none; }
        /* نافذة منبثقة لتسمية الفيديو */
        #video-name-modal {
          display: none;
          position: fixed;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          background: rgba(0, 0, 0, 0.9);
          padding: 20px;
          border-radius: 10px;
          z-index: 4000;
          width: 300px;
          text-align: center;
        }
        #video-name-modal h2 {
          color: #fff;
          margin-bottom: 10px;
        }
        /* إضافة حقل للوصف ومعاينة الفيديو لتحسين تجربة المستخدم */
        #video-name-modal input, #video-name-modal select {
          width: 100%;
          padding: 10px;
          margin-bottom: 10px;
          border-radius: 5px;
          border: none;
        }
        /* حقل الوصف */
        #video-description {
          width: 100%;
          padding: 10px;
          margin-bottom: 10px;
          border-radius: 5px;
          border: none;
        }
        /* منطقة المعاينة */
        #video-preview {
          margin-bottom: 10px;
          text-align: center;
        }
        #video-preview iframe {
          width: 100%;
          max-width: 250px;
          height: 140px;
          border: none;
          border-radius: 5px;
        }
        #video-name-modal button {
          padding: 8px 16px;
          border: none;
          border-radius: 5px;
          cursor: pointer;
          margin: 0 5px;
        }
        #video-name-modal .confirm-btn {
          background: #00d1b2;
          color: #fff;
        }
        #video-name-modal .cancel-btn {
          background: #ff6f61;
          color: #fff;
        }
        /* نافذة منبثقة لإنشاء مجلد جديد */
        #folder-modal {
          display: none;
          position: fixed;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          background: rgba(0, 0, 0, 0.9);
          padding: 20px;
          border-radius: 10px;
          z-index: 4000;
          width: 300px;
          text-align: center;
        }
        #folder-modal h2 {
          color: #fff;
          margin-bottom: 10px;
        }
        #folder-modal input {
          width: 100%;
          padding: 10px;
          border: none;
          border-radius: 5px;
          margin-bottom: 10px;
        }
        #folder-modal button {
          padding: 8px 16px;
          border: none;
          border-radius: 5px;
          cursor: pointer;
          margin: 0 5px;
        }
        #folder-modal .confirm-btn {
          background: #00d1b2;
          color: #fff;
        }
        #folder-modal .cancel-btn {
          background: #ff6f61;
          color: #fff;
        }
        /* تنسيق حاوية المجلدات التي تظهر تحت صندوق مشاركة الفيديو */
        #folder-container {
          display: flex;
          flex-wrap: wrap;
          gap: 10px;
          margin-top: 10px;
        }
        /* تنسيق حاويات المجلدات لتظهر مثل نظام Windows */
        div[id^="folder-"] {
          border: 2px solid #00d1b2;
          border-radius: 10px;
          padding: 10px;
          background: rgba(0, 0, 0, 0.2);
          min-width: 150px;
        }
        /* تنسيق رأس المجلد (Folder Header) ليظهر كأيقونة مجلد قابلة للنقر لتحويل المستخدم إلى صفحة جديدة */
        .folder-header {
          cursor: pointer;
          font-size: 1.2em;
          color: #00d1b2;
          display: flex;
          align-items: center;
          margin-bottom: 10px;
        }
        .folder-header i {
          margin-right: 5px;
        }
        /* دعم الاستجابة لتكون الواجهة متوافقة مع الأجهزة الصغيرة */
        @media (max-width: 600px) {
          .video-share-box input, .video-share-box button {
            width: 100%;
            margin-bottom: 10px;
          }
        }
      </style>
      {% endraw %}
    </head>
    <body>
      <div class="container">
        <div class="sidebar">
          <a href="/home/chat" title="الشات" onclick="showSection('chat', event)">
            <i class="fa-solid fa-comment-dots"></i>
          </a>
          <a href="/home/feed" title="الفيد" onclick="showSection('feed', event)">
            <i class="fa-solid fa-newspaper"></i>
          </a>
          <a href="/home/profile" title="البروفايل" onclick="showSection('profile', event)">
            <i class="fa-solid fa-user"></i>
          </a>
          <!-- أيقونة الفيديوهات المضافة -->
          <a href="/home/videos" title="الفيديوهات" onclick="showSection('videos', event)">
            <i class="fa-solid fa-video"></i>
          </a>
          <a href="/home/computer" class="computer-icon" title="الكمبيوتر" onclick="showSection('computer', event)">
            <i class="fa-solid fa-desktop"></i>
          </a>
          <!-- أيقونة مستخدمي الموقع -->
          <a href="/users" class="user-icon" title="مستخدمو الموقع">
            <i class="fa-solid fa-users"></i>
          </a>
          <div class="bottom-icons">
            <a href="/home/trash" class="trash-icon" title="سلة المهملات" onclick="showSection('trash', event)">
              <i class="fa-solid fa-trash"></i>
            </a>
            <a href="#" class="close-icon" title="إغلاق" onclick="window.location.href='/logout'">
              <i class="fa-solid fa-xmark"></i>
            </a>
          </div>
        </div>
        <!-- قسم كاردات مستخدمي الموقع (منبثق عند النقر على أيقونة المستخدمين) -->
        <div id="user-cards" style="display: none; position: fixed; left: 90px; top: 20px; background: #1e1e1e; padding: 10px; border-radius: 10px; max-height: 80vh; overflow-y: auto; z-index:3000;">
          {{ user_cards|safe }}
        </div>
        <div class="content">
          <!-- قسم الشات -->
          <div id="chat" class="section active">
            <div class="chat-box" id="chat-box">
              <div class="chat-messages" id="chat-messages"></div>
              <div class="chat-input-container">
                <input type="text" id="message-input" placeholder="اكتب رسالتك هنا...">
                <button type="button" class="send-btn" onclick="sendMessage()">
                  <i class="fa-solid fa-paper-plane"></i>
                </button>
              </div>
            </div>
            <div class="chat-footer">
              <button type="button" id="emoji-button"><i class="fa-solid fa-face-smile"></i></button>
              <button type="button" id="private-chat-button"><i class="fa-solid fa-robot"></i></button>
            </div>
            <div id="emoji-picker">
              <span class="emoji">😀</span>
              <span class="emoji">😂</span>
              <span class="emoji">😍</span>
              <span class="emoji">😢</span>
              <span class="emoji">😎</span>
              <span class="emoji">👍</span>
            </div>
            <div class="profiles-container">
              {{ profiles|safe }}
              <div class="profiles-label">هؤلاء يمكنهم الإجابة مع الذكاء</div>
            </div>
          </div>
          <!-- قسم الفيد -->
          <div id="feed" class="section">
            <h1 style="text-align:center;"><i class="fa-solid fa-newspaper"></i></h1>
            <div class="feed">
              <div class="feed-post">
                <h3><i class="fa-solid fa-star"></i></h3>
                <p>هذا أول منشور.</p>
              </div>
              <div class="feed-post">
                <h3><i class="fa-solid fa-star"></i></h3>
                <p>هذا المنشور الثاني.</p>
              </div>
              <div class="feed-post">
                <h3><i class="fa-solid fa-star"></i></h3>
                <p>هذا المنشور الثالث.</p>
              </div>
            </div>
          </div>
          <!-- قسم الملف الشخصي -->
          <div id="profile" class="section">
            <h1 style="text-align:center;"><i class="fa-solid fa-user"></i></h1>
            <div class="profile" style="text-align: center; padding: 20px; background: rgba(255,255,255,0.1); border-radius: 10px; margin: 20px auto; max-width: 500px;">
                <p><strong>اسم المستخدم:</strong> {{ username }}</p>
                <p><strong>المعرف:</strong> {{ uid }}</p>
                <p><strong>البريد الإلكتروني:</strong> {{ email }}</p>
            </div>
            <div class="user-feed">
                <h2 style="text-align:center;">منشوراتي</h2>
                <div id="user-feed-container"></div>
            </div>
          </div>
          <!-- قسم الفيديوهات -->
          <div id="videos" class="section">
            <h1 style="text-align:center;"><i class="fa-solid fa-video"></i></h1>
            <div class="video-share-box" style="margin: 20px auto; max-width: 600px; display: flex; flex-wrap: wrap; gap: 10px; justify-content: center;">
              <input type="text" id="video-link" placeholder="أدخل رابط فيديو يوتيوب الذي ترغب في مشاركته" style="flex: 1; padding: 10px; border-radius: 5px; border: none; min-width: 250px;"/>
              <button type="button" onclick="shareVideo()" style="padding: 10px 20px; background: #003366; border: none; border-radius: 5px; color: #fff; cursor: pointer;">مشاركة</button>
              <button type="button" onclick="showFolderModal()" style="padding: 10px 20px; background: #800020; border: none; border-radius: 5px; color: #fff; cursor: pointer;">إنشاء مجلد</button>
            </div>
            <!-- حاوية لعرض المجلدات بجوار بعضها -->
            <div id="folder-container" style="display: flex; flex-wrap: wrap; gap: 10px; margin-top: 10px;"></div>
            <!-- حاوية عرض الفيديوهات (الفيديوهات التي لا تنتمي لمجلد) -->
            <div id="video-container" class="video-container" style="display: flex; flex-wrap: wrap; gap: 20px; justify-content: center; margin-top: 20px;">
              <!-- سيتم إضافة الفيديوهات هنا -->
            </div>
          </div>
          <!-- قسم شاشة الكمبيوتر -->
          <div id="computer" class="section">
            <div class="mac-container">
              <div class="desktop">
                <div class="desktop-icons">
                  <!-- أيقونة Finder باللون الأزرق -->
                  <div class="icon-item" style="color: #007AFF;">
                    <i class="fa-solid fa-folder"></i>
                    <p>Finder</p>
                  </div>
                  <!-- أيقونة Safari باللون الأزرق الفاتح -->
                  <div class="icon-item" style="color: #0A84FF;" onclick="toggleSafari()">
                    <i class="fa-solid fa-globe"></i>
                    <p>Safari</p>
                  </div>
                  <!-- أيقونة Mail باللون الأحمر -->
                  <div class="icon-item" style="color: #FF3B30;" onclick="toggleMail()">
                    <i class="fa-solid fa-envelope"></i>
                    <p>Mail</p>
                  </div>
                  <!-- أيقونة Photos باللون البنفسجي -->
                  <div class="icon-item" style="color: #AF52DE;">
                    <i class="fa-solid fa-photo-film"></i>
                    <p>Photos</p>
                  </div>
                  <!-- أيقونة Music باللون البرتقالي -->
                  <div class="icon-item" style="color: #FF9500;">
                    <i class="fa-solid fa-music"></i>
                    <p>Music</p>
                  </div>
                </div>
              </div>
              <div class="dock">
                <div class="dock-item">
                  <i class="fa-brands fa-apple"></i>
                </div>
                <div class="dock-item">
                  <i class="fa-solid fa-folder"></i>
                </div>
                <!-- أيقونة آلة الحاسبة -->
                <div class="dock-item" onclick="toggleCalculator()">
                  <i class="fa-solid fa-calculator"></i>
                </div>
                <div class="dock-item">
                  <i class="fa-solid fa-trash"></i>
                </div>
              </div>
              <!-- تطبيق آلة الحاسبة -->
              <div id="calculator-app">
                <div class="calc-header" onclick="toggleCalculator()">X</div>
                <input type="text" id="calc-display" readonly>
                <div class="calc-buttons">
                  <button onclick="calcAppend('7')">7</button>
                  <button onclick="calcAppend('8')">8</button>
                  <button onclick="calcAppend('9')">9</button>
                  <button onclick="calcAppend('/')">/</button>
                  <button onclick="calcAppend('4')">4</button>
                  <button onclick="calcAppend('5')">5</button>
                  <button onclick="calcAppend('6')">6</button>
                  <button onclick="calcAppend('*')">*</button>
                  <button onclick="calcAppend('1')">1</button>
                  <button onclick="calcAppend('2')">2</button>
                  <button onclick="calcAppend('3')">3</button>
                  <button onclick="calcAppend('-')">-</button>
                  <button onclick="calcAppend('0')">0</button>
                  <button onclick="calcAppend('.')">.</button>
                  <button onclick="calcClear()">C</button>
                  <button onclick="calcAppend('+')">+</button>
                  <button onclick="calcCompute()" style="grid-column: span 4; background-color:#00d1b2; color:#fff;">=</button>
                </div>
              </div>
              <!-- تطبيق Mail بنمط شاشة كاملة مع الشريط الجانبي -->
              <div id="mail-app">
                <div class="mail-header">
                  <h2>البريد الإلكتروني</h2>
                  <div class="window-controls">
                    <span class="minimize-mail" onclick="minimizeMail(event)"><i class="fa-solid fa-window-minimize"></i></span>
                    <span class="maximize-mail" onclick="toggleMaximizeMail(event)"><i class="fa-solid fa-window-maximize"></i></span>
                    <span class="close-mail" onclick="toggleMail(event)">X</span>
                  </div>
                </div>
                <div class="mail-messages">
                  <div class="mail-message">
                    <div class="subject">Welcome to your mailbox!</div>
                    <div class="snippet">Thank you for joining us. Enjoy your experience.</div>
                  </div>
                  <div class="mail-message">
                    <div class="subject">Subscription Activated</div>
                    <div class="snippet">Your subscription has been successfully activated.</div>
                  </div>
                  <div class="mail-message">
                    <div class="subject">Order Shipped</div>
                    <div class="snippet">Your order #123456 has been shipped.</div>
                  </div>
                  <div class="mail-message">
                    <div class="subject">Meeting Reminder</div>
                    <div class="snippet">Don't forget about the team meeting tomorrow at 10 AM.</div>
                  </div>
                  <div class="mail-message">
                    <div class="subject">Happy Birthday!</div>
                    <div class="snippet">Wishing you a wonderful birthday from our team.</div>
                  </div>
                </div>
              </div>
              <!-- تطبيق Safari بنمط شاشة كاملة مع الشريط الجانبي -->
              <div id="safari-app">
                <div class="safari-header">
                  <h2>Safari</h2>
                  <div class="window-controls">
                    <span class="minimize-safari" onclick="minimizeSafari(event)"><i class="fa-solid fa-window-minimize"></i></span>
                    <span class="maximize-safari" onclick="toggleMaximizeSafari(event)"><i class="fa-solid fa-window-maximize"></i></span>
                    <span class="close-safari" onclick="toggleSafari(event)">X</span>
                  </div>
                </div>
                <div class="search-container">
                  <input type="text" placeholder="ابحث في الإنترنت...">
                  <button>بحث</button>
                </div>
              </div>
            </div>
          </div>
          <!-- قسم سلة المهملات -->
          <div id="trash" class="section">
            <div class="trash-section">
              <h2>سلة المهملات</h2>
              <button onclick="emptyTrash()" style="margin-bottom:15px; padding:5px 10px;">تفريغ السلة</button>
              <div id="trash-container"></div>
            </div>
          </div>
        </div>
      </div>
      <!-- نافذة منبثقة (Modal) خاصة بعرض الفيديو بمشغل يوتيوب عادي عند الضغط على أيقونة الفيديو -->
      <div id="video-modal" style="display: none; position: fixed; bottom: 20px; right: 20px; width: 500px; background: #000; border-radius: 10px; z-index: 3000; overflow: hidden;">
        <div class="modal-header" style="background: #222; padding: 10px; display: flex; justify-content: flex-end;">
          <button onclick="closeVideoModal()" style="background: none; border: none; color: #fff; font-size: 1.5em; cursor: pointer;">&times;</button>
        </div>
        <iframe id="video-modal-iframe" src="" style="width: 100%; height: 280px; border: none;" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
        <div id="iframe-overlay" style="display: none;"></div>
        <div id="video-custom-controls" style="display: none;"></div>
      </div>
      <!-- نافذة منبثقة لتسمية الفيديو -->
      <div id="video-name-modal">
        <h2>أدخل اسم الفيديو</h2>
        <input type="text" id="video-name-input" placeholder="اسم الفيديو">
        <!-- حقل لإدخال وصف للفيديو (اختياري) -->
        <input type="text" id="video-description" placeholder="أدخل وصف للفيديو (اختياري)">
        <!-- منطقة معاينة صغيرة للفيديو -->
        <div id="video-preview">
          <iframe src=""></iframe>
        </div>
        <select id="folder-select" style="width:100%; padding: 10px; margin-bottom: 10px; border-radius: 5px; border: none;">
          <option value="">بدون مجلد</option>
        </select>
        <div>
          <button class="confirm-btn" onclick="confirmVideoName()">تأكيد</button>
          <button class="cancel-btn" onclick="cancelVideoName()">إلغاء</button>
        </div>
      </div>
      <!-- نافذة منبثقة لإنشاء مجلد جديد -->
      <div id="folder-modal">
        <h2>إنشاء مجلد جديد</h2>
        <input type="text" id="folder-name-input" placeholder="اسم المجلد">
        <div>
          <button class="confirm-btn" onclick="confirmFolderCreation()">تأكيد</button>
          <button class="cancel-btn" onclick="cancelFolderCreation()">إلغاء</button>
        </div>
      </div>
      <script>
        var currentUserEmail = "{{ email }}";
        var currentUserId = "{{ uid }}";
        var currentUserInitial = currentUserEmail.charAt(0).toUpperCase();
        var currentUserAvatar = "{{ avatar }}";
        var currentUsername = "{{ username }}";
        var clonedPosts = {};
        var trashItems = {}; // لتخزين المنشورات المنقولة إلى سلة المهملات
        var privateChatMode = false;
        var currentVideoSrc = "";
        var folders = []; // مصفوفة لتخزين أسماء المجلدات
      </script>
      {% raw %}
      <script>
        // دالة لتحديث قائمة المجلدات في نافذة تسمية الفيديو
        function updateFolderSelect() {
          var folderSelect = document.getElementById('folder-select');
          folderSelect.innerHTML = '<option value="">بدون مجلد</option>';
          folders.forEach(function(folder){
            var opt = document.createElement('option');
            opt.value = folder;
            opt.textContent = folder;
            folderSelect.appendChild(opt);
          });
        }
        // دالة للتبديل بين الأقسام وتحديث الرابط في المتصفح
        function showSection(sectionId, event) {
          if (event) {
            event.preventDefault();
          }
          
          // إخفاء جميع الأقسام
          var sections = document.getElementsByClassName('section');
          for (var i = 0; i < sections.length; i++) {
            sections[i].classList.remove('active');
          }
          
          // إظهار القسم المحدد
          var selectedSection = document.getElementById(sectionId);
          if (selectedSection) {
            selectedSection.classList.add('active');
          }
          
          // تحديث الرابط في المتصفح
          var newUrl = '/home/' + sectionId;
          history.pushState({section: sectionId}, '', newUrl);
        }

        // التأكد من إظهار القسم الصحيح عند تحميل الصفحة
        document.addEventListener('DOMContentLoaded', function() {
          var currentSection = '{{ section }}';
          showSection(currentSection);
        });

        // التعامل مع أزرار التنقل في المتصفح
        window.addEventListener('popstate', function(event) {
          if (event.state && event.state.section) {
            showSection(event.state.section);
          }
        });

        // دالة للتوجه إلى صفحة المجلد عند النقر على رأسه
        function goToFolder(folderName) {
          window.location.href = '/folder/' + encodeURIComponent(folderName);
        }
        document.getElementById('private-chat-button').addEventListener('click', function(){
          privateChatMode = !privateChatMode;
          this.style.color = privateChatMode ? "var(--accent-color)" : "var(--primary-color)";
          console.log(privateChatMode ? "تم تفعيل الدردشة الخاصة مع الذكاء." : "تم إلغاء تفعيل الدردشة الخاصة.");
        });
        function sendMessage() {
          var input = document.getElementById('message-input');
          var messageText = input.value.trim();
          if (messageText === '') return;
          addMessage('user', messageText);
          input.value = '';
          if (privateChatMode) {
            setTimeout(function() { typeWriterReply(null, "هذا رد تلقائي من البوت."); }, 1000);
          } else {
            var feedContainer = document.querySelector('#feed .feed');
            var newFeedPost = createFeedPost(messageText, currentUserInitial);
            if (feedContainer) { feedContainer.insertBefore(newFeedPost, feedContainer.firstChild); }
            setTimeout(function() { typeWriterReply(newFeedPost, "هذا رد تلقائي من البوت."); }, 1000);
          }
        }
        function addMessage(sender, text) {
          var chatMessages = document.getElementById('chat-messages');
          var messageDiv = document.createElement('div');
          if (sender === "user") { messageDiv.classList.add("user-message"); }
          else { messageDiv.style.backgroundColor = "#333"; messageDiv.style.alignSelf = "flex-start"; }
          messageDiv.style.marginBottom = "10px";
          messageDiv.style.padding = "10px";
          messageDiv.style.borderRadius = "8px";
          messageDiv.style.maxWidth = "80%";
          messageDiv.textContent = text;
          chatMessages.appendChild(messageDiv);
          chatMessages.scrollTop = chatMessages.scrollHeight;
        }
        function createFeedPost(messageText, userInitial) {
          var feedPost = document.createElement('div');
          feedPost.className = 'feed-post';
          feedPost.dataset.user = currentUserEmail;
          var postId = 'post-' + Date.now();
          feedPost.id = postId;

          // تحديث رأس المنشور ليشمل صورة المستخدم واسمه
          var postHeader = document.createElement('div');
          postHeader.className = 'post-header';
          
          // إضافة صورة المستخدم
          var avatarContainer = document.createElement('div');
          avatarContainer.className = 'post-avatar-container';
          if (currentUserAvatar && currentUserAvatar !== "") {
              var avatarImg = document.createElement('img');
              avatarImg.src = currentUserAvatar;
              avatarImg.className = 'post-avatar-img';
              avatarContainer.appendChild(avatarImg);
          } else {
              var defaultAvatar = document.createElement('div');
              defaultAvatar.className = 'post-avatar';
              defaultAvatar.textContent = userInitial;
              avatarContainer.appendChild(defaultAvatar);
          }
          postHeader.appendChild(avatarContainer);

          // إضافة معلومات المستخدم
          var postInfo = document.createElement('div');
          postInfo.className = 'post-info';
          var userNameSpan = document.createElement('span');
          userNameSpan.className = 'post-user';
          userNameSpan.textContent = currentUsername; // استخدام اسم المستخدم بدلاً من البريد الإلكتروني
          var timeSpan = document.createElement('span');
          timeSpan.className = 'post-time';
          timeSpan.textContent = new Date().toLocaleTimeString('ar-SA');
          postInfo.appendChild(userNameSpan);
          postInfo.appendChild(timeSpan);
          postHeader.appendChild(postInfo);

          feedPost.appendChild(postHeader);

          var postContent = document.createElement('div');
          postContent.className = 'post-content';
          var postParagraph = document.createElement('p');
          postParagraph.textContent = messageText;
          postContent.appendChild(postParagraph);
          feedPost.appendChild(postContent);
          var actionsDiv = document.createElement('div');
          actionsDiv.className = 'post-actions';
          var editBtn = document.createElement('button');
          editBtn.innerHTML = '<i class="fa-solid fa-edit"></i>';
          editBtn.onclick = function() { editPost(postId); };
          var deleteBtn = document.createElement('button');
          deleteBtn.innerHTML = '<i class="fa-solid fa-trash"></i>';
          deleteBtn.onclick = function() { deletePost(postId); };
          actionsDiv.appendChild(editBtn);
          actionsDiv.appendChild(deleteBtn);
          feedPost.appendChild(actionsDiv);
          var supportDiv = document.createElement('div');
          supportDiv.className = 'post-support';
          var supportBtn = document.createElement('button');
          supportBtn.innerHTML = '<i class="fa-solid fa-heart"></i>';
          supportBtn.className = 'support-btn';
          supportBtn.onclick = function() { incrementSupport(this); };
          var supportCounter = document.createElement('span');
          supportCounter.className = 'support-counter';
          supportCounter.textContent = "0";
          supportDiv.appendChild(supportBtn);
          supportDiv.appendChild(supportCounter);
          feedPost.appendChild(supportDiv);
          var postComments = document.createElement('div');
          postComments.className = 'post-comments';
          feedPost.appendChild(postComments);
          var commentInputDiv = document.createElement('div');
          commentInputDiv.className = 'comment-input';
          var commentInput = document.createElement('input');
          commentInput.type = 'text';
          commentInput.placeholder = 'اكتب تعليقك هنا...';
          commentInput.className = 'comment-text';
          commentInputDiv.appendChild(commentInput);
          var commentButton = document.createElement('button');
          commentButton.innerHTML = '<i class="fa-solid fa-paper-plane"></i>';
          commentButton.onclick = function() { postComment(this); };
          commentInputDiv.appendChild(commentButton);
          feedPost.appendChild(commentInputDiv);
          var clonedPost = feedPost.cloneNode(true);
          clonedPost.id = postId + '-clone';
          var userFeedContainer = document.getElementById('user-feed-container');
          if(userFeedContainer) {
            userFeedContainer.insertBefore(clonedPost, userFeedContainer.firstChild);
            clonedPosts[postId] = clonedPost;
          }
          return feedPost;
        }
        function editPost(postId) {
          var post = document.getElementById(postId);
          var contentPara = post.querySelector('.post-content p');
          var currentText = contentPara.textContent;
          var newText = prompt("تعديل المنشور:", currentText);
          if(newText !== null) {
            contentPara.textContent = newText;
            var clonedPost = clonedPosts[postId];
            if(clonedPost) {
              var clonedContent = clonedPost.querySelector('.post-content p');
              if(clonedContent) { clonedContent.textContent = newText; }
            }
          }
        }
        function deletePost(postId) {
          var post = document.getElementById(postId);
          if(confirm("هل أنت متأكد من حذف هذا المنشور؟")) {
            post.remove();
            if(clonedPosts[postId]) {
              clonedPosts[postId].remove();
              delete clonedPosts[postId];
            }
            var trashContainer = document.getElementById('trash-container');
            trashContainer.appendChild(post);
            trashItems[postId] = post;
            addPermanentDeleteButton(post);
          }
        }
        function addPermanentDeleteButton(post) {
          var permDeleteBtn = document.createElement('button');
          permDeleteBtn.innerHTML = '<i class="fa-solid fa-trash"></i> حذف نهائي';
          permDeleteBtn.style.marginLeft = '10px';
          permDeleteBtn.onclick = function() { permanentDelete(post.id); };
          var actionsDiv = post.querySelector('.post-actions');
          if(actionsDiv) { actionsDiv.appendChild(permDeleteBtn); }
          else { post.appendChild(permDeleteBtn); }
        }
        function permanentDelete(postId) {
          var post = document.getElementById(postId);
          if(post) { post.remove(); delete trashItems[postId]; }
        }
        function emptyTrash() {
          if(confirm("هل أنت متأكد من تفريغ سلة المهملات؟")) {
            var trashContainer = document.getElementById('trash-container');
            while(trashContainer.firstChild) { trashContainer.firstChild.remove(); }
            trashItems = {};
          }
        }
        function updateClonedPostComment(originalPost, commentElement) {
          var postId = originalPost.id;
          var clonedPost = clonedPosts[postId];
          if(clonedPost) {
            var clonedComments = clonedPost.querySelector('.post-comments');
            if(clonedComments) {
              var clonedComment = commentElement.cloneNode(true);
              clonedComments.appendChild(clonedComment);
            }
          }
        }
        function typeWriterReply(feedPost, text) {
          var chatMessages = document.getElementById('chat-messages');
          var botMessageDiv = document.createElement('div');
          botMessageDiv.style.marginBottom = "10px";
          botMessageDiv.style.padding = "10px";
          botMessageDiv.style.borderRadius = "8px";
          botMessageDiv.style.maxWidth = "80%";
          botMessageDiv.style.backgroundColor = "#333";
          botMessageDiv.style.alignSelf = "flex-start";
          chatMessages.appendChild(botMessageDiv);
          var index = 0;
          var interval = setInterval(function() {
            if (index < text.length) {
              botMessageDiv.textContent += text.charAt(index);
              index++;
              chatMessages.scrollTop = chatMessages.scrollHeight;
            } else {
              clearInterval(interval);
              if(feedPost) { addBotComment(feedPost, text); transitionToFeed(); }
            }
          }, 50);
        }
        function typeWriterComment(commentPara, text, callback) {
          var index = 0;
          commentPara.textContent = "";
          var interval = setInterval(function(){
            if (index < text.length) { commentPara.textContent += text.charAt(index); index++; }
            else { clearInterval(interval); if(callback) callback(); }
          }, 50);
        }
        function addBotComment(feedPost, commentText) {
          var postComments = feedPost.querySelector('.post-comments');
          var commentDiv = document.createElement('div');
          commentDiv.className = 'comment';
          var botReply = document.createElement('p');
          botReply.textContent = "هذا رد تلقائي من البوت.";
          commentDiv.appendChild(botReply);
          var aiLine = document.createElement('p');
          aiLine.className = 'ai-label';
          aiLine.textContent = "من الذكاء الاصطناعي";
          commentDiv.appendChild(aiLine);
          postComments.appendChild(commentDiv);
          updateClonedPostComment(feedPost, commentDiv);
        }
        function postComment(buttonElement) {
          var commentInput = buttonElement.parentElement.querySelector('.comment-text');
          var commentText = commentInput.value.trim();
          if(commentText === '') return;
          var feedPost = buttonElement.parentElement.parentElement;
          var postComments = feedPost.querySelector('.post-comments');
          var commentDiv = document.createElement('div');
          commentDiv.className = 'comment user-comment';
          var commentPara = document.createElement('p');
          commentPara.textContent = "";
          commentDiv.appendChild(commentPara);
          var authorPara = document.createElement('p');
          var authorLink = document.createElement('a');
          authorLink.href = "/profile/" + currentUserId;
          authorLink.className = 'reply-author';
          authorLink.textContent = "من " + currentUserEmail;
          authorPara.appendChild(authorLink);
          commentDiv.appendChild(authorPara);
          postComments.appendChild(commentDiv);
          commentInput.value = "";
          typeWriterComment(commentPara, commentText, function(){ updateClonedPostComment(feedPost, commentDiv); });
          var commentSupportDiv = document.createElement('div');
          commentSupportDiv.className = 'comment-support';
          var commentSupportBtn = document.createElement('button');
          commentSupportBtn.innerHTML = '<i class="fa-solid fa-heart"></i>';
          commentSupportBtn.className = 'support-btn';
          commentSupportBtn.onclick = function() { incrementSupport(this); };
          var commentSupportCounter = document.createElement('span');
          commentSupportCounter.className = 'support-counter';
          commentSupportCounter.textContent = "0";
          commentSupportDiv.appendChild(commentSupportBtn);
          commentSupportDiv.appendChild(commentSupportCounter);
          commentDiv.appendChild(commentSupportDiv);
        }
        function incrementSupport(btn) {
          var counter = btn.nextElementSibling;
          var count = parseInt(counter.textContent) || 0;
          count++;
          counter.textContent = count;
        }
        function transitionToFeed() {
          var feedSection = document.getElementById('feed');
          feedSection.classList.add('magic');
          setTimeout(function() { showSection('feed'); feedSection.classList.remove('magic'); }, 1000);
        }
        document.getElementById('emoji-button').addEventListener('click', function() {
          var picker = document.getElementById('emoji-picker');
          picker.style.display = (picker.style.display === 'none' || picker.style.display === '') ? 'block' : 'none';
        });
        var emojis = document.querySelectorAll('#emoji-picker .emoji');
        emojis.forEach(function(emoji) {
          emoji.addEventListener('click', function() {
            var input = document.getElementById('message-input');
            input.value += this.textContent;
            document.getElementById('emoji-picker').style.display = 'none';
            input.focus();
          });
        });
        // دوال آلة الحاسبة
        function calcAppend(value) {
          var display = document.getElementById('calc-display');
          display.value += value;
        }
        function calcClear() {
          var display = document.getElementById('calc-display');
          display.value = "";
        }
        function calcCompute() {
          var display = document.getElementById('calc-display');
          try { display.value = eval(display.value); }
          catch (e) { display.value = "خطأ"; }
        }
        function toggleCalculator() {
          var calcApp = document.getElementById('calculator-app');
          if (calcApp.style.display === 'none' || calcApp.style.display === '') { calcApp.style.display = 'block'; }
          else { calcApp.style.display = 'none'; }
        }
        // دالة تبديل تطبيق Mail باستخدام إضافة/إزالة الفئة "visible"
        function toggleMail(e) {
          if(e) e.stopPropagation();
          var mailApp = document.getElementById('mail-app');
          if (mailApp.classList.contains('visible')) { mailApp.classList.remove('visible'); }
          else { mailApp.classList.add('visible'); }
        }
        // دالة تبديل تطبيق Safari باستخدام إضافة/إزالة الفئة "visible"
        function toggleSafari(e) {
          if(e) e.stopPropagation();
          var safariApp = document.getElementById('safari-app');
          if (safariApp.classList.contains('visible')) { safariApp.classList.remove('visible'); }
          else { safariApp.classList.add('visible'); }
        }
        // دوال التحكم بحجم نافذة Mail
        function minimizeMail(e) {
          e.stopPropagation();
          var mailApp = document.getElementById('mail-app');
          mailApp.classList.add('minimized');
        }
        function toggleMaximizeMail(e) {
          e.stopPropagation();
          var mailApp = document.getElementById('mail-app');
          if (mailApp.classList.contains('minimized')) { mailApp.classList.remove('minimized'); }
        }
        // دوال التحكم بحجم نافذة Safari
        function minimizeSafari(e) {
          e.stopPropagation();
          var safariApp = document.getElementById('safari-app');
          safariApp.classList.add('minimized');
        }
        function toggleMaximizeSafari(e) {
          e.stopPropagation();
          var safariApp = document.getElementById('safari-app');
          if (safariApp.classList.contains('minimized')) { safariApp.classList.remove('minimized'); }
        }
        function submitBubblePost(){
          var text = document.getElementById('bubble-text').value.trim();
          var duration = parseInt(document.getElementById('bubble-duration').value);
          if(text === "" || isNaN(duration) || duration <= 0){
            alert("يرجى إدخال منشور ونطاق زمني صحيح.");
            return;
          }
          var bubblePost = document.createElement('div');
          bubblePost.className = 'bubble-post';
          bubblePost.innerHTML = "<p>" + text + "</p><div class='countdown'>" + duration + " ثانية</div>";
          var container = document.getElementById('bubble-post-container');
          container.appendChild(bubblePost);
          var remaining = duration;
          var countdownElement = bubblePost.querySelector('.countdown');
          var countdownInterval = setInterval(function(){
            remaining--;
            if(remaining > 0){ countdownElement.textContent = remaining + " ثانية"; }
            else {
              clearInterval(countdownInterval);
              bubblePost.classList.add('explode');
              setTimeout(function(){ bubblePost.remove(); }, 1000);
            }
          }, 1000);
        }
        // دالة مشاركة فيديو يوتيوب باستخدام تعبير نمطي للتحقق من الرابط
        function shareVideo(){
          var videoLinkInput = document.getElementById('video-link');
          var link = videoLinkInput.value.trim();
          if(link === ""){
            alert("يرجى إدخال رابط فيديو يوتيوب.");
            return;
          }
          // استخدام تعبير نمطي للتحقق من الرابط واستخراج معرف الفيديو
          var regex = /^(?:https?:\/\/)?(?:www\.)?(?:youtube\.com\/(?:watch\\?v=|embed\/)|youtu\.be\/)([a-zA-Z0-9_-]{11})(?:\\S+)?$/;
          var match = link.match(regex);
          if(match && match[1]){
            var videoId = match[1];
            var videoSrc = "https://www.youtube.com/embed/" + videoId + "?modestbranding=1&controls=1&rel=0&showinfo=0";
            showVideoNameModal(videoSrc);
            videoLinkInput.value = "";
          } else {
            alert("الرابط المدخل غير صالح. الرجاء التأكد من أنه رابط فيديو يوتيوب صحيح.");
            return;
          }
        }
        // دالة فتح نافذة تسمية الفيديو مع عرض معاينة ووصف اختياري
        function showVideoNameModal(videoSrc) {
          currentVideoSrc = videoSrc;
          document.getElementById('video-name-input').value = "";
          document.getElementById('video-description').value = "";
          updateFolderSelect();
          // تعيين مصدر المعاينة في النافذة المنبثقة
          document.querySelector('#video-preview iframe').src = videoSrc;
          document.getElementById('video-name-modal').style.display = "block";
        }
        // دالة تأكيد اسم الفيديو واختيار المجلد مع حفظ البيانات في قاعدة البيانات
        function confirmVideoName() {
          var videoName = document.getElementById('video-name-input').value.trim();
          if(videoName === ""){
            alert("يرجى إدخال اسم للفيديو.");
            return;
          }
          var videoDescription = document.getElementById('video-description').value.trim();
          var folder = document.getElementById('folder-select').value;
          // حفظ بيانات الفيديو في قاعدة البيانات (مثال باستخدام Fetch)
          saveVideoData(currentVideoSrc, videoName, folder, videoDescription);
          createVideoItem(currentVideoSrc, videoName, folder, videoDescription);
          document.getElementById('video-name-modal').style.display = "none";
        }
        // دالة إلغاء تسمية الفيديو
        function cancelVideoName() {
          document.getElementById('video-name-modal').style.display = "none";
        }
        // دالة إرسال بيانات الفيديو إلى السيرفر لحفظها (يمكن تعديلها لتناسب Firestore)
        function saveVideoData(videoSrc, videoName, folder, description) {
          fetch('/save_video', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ videoSrc: videoSrc, videoName: videoName, folder: folder, description: description })
          })
          .then(response => response.json())
          .then(data => {
            if(data.success){
              console.log("تم حفظ بيانات الفيديو بنجاح.");
            } else {
              console.log("حدث خطأ أثناء حفظ بيانات الفيديو.");
            }
          })
          .catch(err => {
            console.error("خطأ في الاتصال بالسيرفر:", err);
          });
        }
        // دالة فتح نافذة إنشاء مجلد جديد
        function showFolderModal(){
          document.getElementById('folder-name-input').value = "";
          document.getElementById('folder-modal').style.display = "block";
        }
        // دالة تأكيد إنشاء المجلد الجديد مع تعديل الحدث للنقل إلى صفحة المجلد
        function confirmFolderCreation(){
          var folderName = document.getElementById('folder-name-input').value.trim();
          if(folderName === ""){
            alert("يرجى إدخال اسم للمجلد.");
            return;
          }
          // إذا كان المجلد غير موجود بالفعل
          if(folders.indexOf(folderName) === -1){
            folders.push(folderName);
            // إنشاء العنصر الخاص بالمجلد وإضافته إلى حاوية المجلدات (folder-container)
            var folderContainerDiv = document.getElementById('folder-container');
            var folderId = "folder-" + folderName;
            if(!document.getElementById(folderId)){
              var folderDiv = document.createElement('div');
              folderDiv.id = folderId;
              // تنسيق المجلد لعرضه كأيقونة مع تأثير hover
              folderDiv.style.border = "2px solid #00d1b2";
              folderDiv.style.borderRadius = "10px";
              folderDiv.style.padding = "10px";
              folderDiv.style.background = "rgba(0, 0, 0, 0.2)";
              folderDiv.style.minWidth = "150px";
              // إنشاء رأس المجلد (Folder Header) مع عرض عدد الفيديوهات (ابتدائي 0)
              var folderHeader = document.createElement('div');
              folderHeader.className = "folder-header";
              folderHeader.innerHTML = '<i class="fa-solid fa-folder"></i> ' + folderName + ' (0)';
              // عند النقر على رأس المجلد، يتم توجيه المستخدم إلى صفحة المجلد الجديدة
              folderHeader.addEventListener('click', function(){
                goToFolder(folderName);
              });
              // إنشاء حاوية لمحتويات المجلد (لن تُستخدم هنا لعرض المحتويات داخل نفس الصفحة)
              var folderContent = document.createElement('div');
              folderContent.className = "folder-content";
              folderContent.style.display = "none";
              folderDiv.appendChild(folderHeader);
              folderDiv.appendChild(folderContent);
              folderContainerDiv.appendChild(folderDiv);
            }
          }
          updateFolderSelect();
          document.getElementById('folder-modal').style.display = "none";
        }
        // دالة إلغاء إنشاء المجلد
        function cancelFolderCreation(){
          document.getElementById('folder-modal').style.display = "none";
        }
        // دالة تحديث عدد الفيديوهات داخل مجلد معين
        function updateFolderCount(folderName) {
          var folderDiv = document.getElementById("folder-" + folderName);
          if(folderDiv) {
            var folderHeader = folderDiv.querySelector('.folder-header');
            var folderContent = folderDiv.querySelector('.folder-content');
            var count = folderContent.children.length;
            folderHeader.innerHTML = '<i class="fa-solid fa-folder"></i> ' + folderName + ' (' + count + ')';
          }
        }
        // دالة إنشاء عنصر الفيديو (أيقونة مع اسم الفيديو ووصفه) وتنظيمه داخل مجلد إن وجد
        function createVideoItem(videoSrc, videoName, folder, description) {
          var videoContainer = document.getElementById('video-container');
          var videoItem = document.createElement('div');
          videoItem.style.textAlign = "center";
          // إنشاء أيقونة الفيديو
          var videoIcon = document.createElement('div');
          videoIcon.className = 'video-icon';
          videoIcon.style.width = "120px";
          videoIcon.style.height = "90px";
          videoIcon.style.background = "#000";
          videoIcon.style.borderRadius = "10px";
          videoIcon.style.overflow = "hidden";
          videoIcon.style.position = "relative";
          videoIcon.style.cursor = "pointer";
          videoIcon.style.boxShadow = "0 4px 10px rgba(0,0,0,0.3)";
          var playIcon = document.createElement('i');
          playIcon.className = 'fa-solid fa-play';
          playIcon.style.position = "absolute";
          playIcon.style.top = "50%";
          playIcon.style.left = "50%";
          playIcon.style.transform = "translate(-50%, -50%)";
          playIcon.style.color = "#fff";
          playIcon.style.fontSize = "2em";
          videoIcon.appendChild(playIcon);
          videoIcon.dataset.src = videoSrc;
          videoIcon.onclick = function(){
            openVideoModal(this.dataset.src);
          };
          videoItem.appendChild(videoIcon);
          // إضافة اسم الفيديو تحت الأيقونة
          var nameElement = document.createElement('p');
          nameElement.textContent = videoName;
          nameElement.style.color = "#fff";
          nameElement.style.marginTop = "5px";
          videoItem.appendChild(nameElement);
          // إذا وُجد وصف للفيديو، إضافته
          if(description && description.trim() !== ""){
            var descElement = document.createElement('p');
            descElement.textContent = description;
            descElement.style.color = "#ccc";
            descElement.style.fontSize = "0.9em";
            videoItem.appendChild(descElement);
          }
          // إضافة زر تعديل للفيديو
          var editBtn = document.createElement('button');
          editBtn.textContent = "تعديل";
          editBtn.style.marginTop = "5px";
          editBtn.onclick = function(){
            var newName = prompt("أدخل اسم جديد للفيديو", videoName);
            if(newName !== null && newName.trim() !== ""){
              nameElement.textContent = newName;
              // TODO: إرسال تحديث إلى السيرفر لتعديل بيانات الفيديو
            }
          };
          videoItem.appendChild(editBtn);
          // إذا تم اختيار مجلد، يتم وضع العنصر داخل المجلد المُخصص الموجود في folder-container
          if(folder && folder !== ""){
            var folderId = "folder-" + folder;
            var folderContainer = document.getElementById(folderId);
            if(folderContainer){
              var folderContent = folderContainer.querySelector('.folder-content');
              folderContent.appendChild(videoItem);
              updateFolderCount(folder);
            } else {
              videoContainer.appendChild(videoItem);
            }
          } else {
            // بدون مجلد، يتم إضافته مباشرة إلى الحاوية الرئيسية للفيديوهات
            videoContainer.appendChild(videoItem);
          }
        }
        // دالة فتح النافذة المنبثقة لعرض الفيديو باستخدام مشغل يوتيوب الافتراضي
        function openVideoModal(src){
          var modal = document.getElementById('video-modal');
          var iframe = document.getElementById('video-modal-iframe');
          iframe.src = src;
          modal.style.display = "block";
        }
        // دالة إغلاق النافذة المنبثقة لعرض الفيديو
        function closeVideoModal(){
          var modal = document.getElementById('video-modal');
          var iframe = document.getElementById('video-modal-iframe');
          iframe.src = "";
          modal.style.display = "none";
        }
        function transitionToFeed() {
          var feedSection = document.getElementById('feed');
          feedSection.classList.add('magic');
          setTimeout(function() { showSection('feed'); feedSection.classList.remove('magic'); }, 1000);
        }
        // دالة لتبديل عرض كاردات مستخدمي الموقع في الشريط الجانبي
        function toggleUserCards(){
          var cards = document.getElementById('user-cards');
          if(cards.style.display === 'none' || cards.style.display === ''){
            cards.style.display = 'block';
          } else {
            cards.style.display = 'none';
          }
        }
      </script>
      {% endraw %}
    </body>
    </html>
    """
    return render_template_string(html, uid=current_uid, email=current_email, username=current_username, avatar=current_avatar, profiles=profiles_html, user_cards=user_cards_html)














# =====================================================================
# صفحة عرض محتويات مجلد محدد (مع الشريط الجانبي)
# =====================================================================
@app.route('/folder/<folder_name>', endpoint='folder_page_alt')
def folder_page_alt(folder_name):
    if 'user' not in session:
        return redirect(url_for('welcome'))
    current_uid = session['user']
    try:
        videos_ref = db.collection("videos").where("user", "==", current_uid).where("folder", "==", folder_name).stream()
        videos = [video.to_dict() for video in videos_ref]
    except Exception as e:
        videos = []
    try:
        user = auth.get_user(current_uid)
        doc_ref = db.collection("users").document(current_uid)
        doc = doc_ref.get()
        if doc.exists:
            user_data = doc.to_dict()
            current_email = user_data.get("email", "غير متوفر")
        else:
            current_email = "غير متوفر"
    except Exception as e:
        current_email = f"خطأ: {e}"
    html = """
    <!DOCTYPE html>
    <html lang="ar">
    <head>
      <meta charset="UTF-8">
      <title>مجلد: {{ folder_name }}</title>
      <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap" rel="stylesheet">
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
      {% raw %}
      <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
          font-family: 'Poppins', sans-serif;
          background: #141e30;
          color: #f0f0f0;
          display: flex;
          overflow: hidden;
        }
        .sidebar {
          width: 80px;
          background: #1e1e1e;
          padding: 20px 5px;
          display: flex;
          flex-direction: column;
          align-items: center;
          border-right: 1px solid #333;
        }
        .sidebar a {
          color: #00d1b2;
          text-decoration: none;
          margin: 15px 0;
          font-size: 1.8em;
          transition: color 0.3s;
        }
        .sidebar a:hover { color: #f39c12; }
        .content {
          flex: 1;
          padding: 20px;
          overflow-y: auto;
          animation: fadeIn 0.8s ease-in-out;
        }
        @keyframes fadeIn {
          from { opacity: 0; }
          to { opacity: 1; }
        }
        .folder-header-page {
          font-size: 2em;
          margin-bottom: 20px;
          text-align: center;
        }
        .video-grid {
          display: flex;
          flex-wrap: wrap;
          gap: 20px;
          margin-top: 20px;
        }
        .video-item {
          background: rgba(255,255,255,0.1);
          padding: 15px;
          border-radius: 10px;
          text-align: center;
          transition: transform 0.3s, box-shadow 0.3s;
        }
        .video-item:hover {
          transform: scale(1.05);
          box-shadow: 0 8px 20px rgba(0,0,0,0.5);
        }
        .video-item img {
          width: 120px;
          height: 90px;
          border-radius: 10px;
          cursor: pointer;
        }
        .video-item p {
          font-size: 0.9em;
          color: #ccc;
          margin-top: 5px;
        }
        .back-btn {
          display: inline-block;
          margin-bottom: 20px;
          padding: 10px 20px;
          background: #00d1b2;
          border: none;
          border-radius: 5px;
          color: #fff;
          text-decoration: none;
          transition: background 0.3s;
        }
        .back-btn:hover { background: #008a80; }
      </style>
      {% endraw %}
    </head>
    <body>
      <div class="sidebar">
         <a href="/home" title="الرئيسية"><i class="fa-solid fa-house"></i></a>
         <a href="/profile/{{ current_uid }}" title="البروفايل"><i class="fa-solid fa-user"></i></a>
         <a href="/bubble" title="المنشورات الفقاعية"><i class="fa-solid fa-comment-dots"></i></a>
         <a href="/logout" title="تسجيل الخروج"><i class="fa-solid fa-xmark"></i></a>
      </div>
      <div class="content">
         <a href="/home" class="back-btn">رجوع للرئيسية</a>
         <div class="folder-header-page">محتويات المجلد: {{ folder_name }}</div>
         <div class="video-grid">
            {{ videos_html|safe }}
         </div>
      </div>
      {% raw %}
      <script>
         function openVideoModal(src) {
           var modal = document.getElementById('video-modal');
           var iframe = document.getElementById('video-modal-iframe');
           iframe.src = src;
           modal.style.display = "block";
         }
         function closeVideoModal() {
           var modal = document.getElementById('video-modal');
           var iframe = document.getElementById('video-modal-iframe');
           iframe.src = "";
           modal.style.display = "none";
         }
      </script>
      {% endraw %}
    </body>
    </html>
    """
    # بناء HTML لعرض الفيديوهات داخل صفحة المجلد
    videos_html = ""
    try:
        videos = db.collection("videos").where("user", "==", current_uid).where("folder", "==", folder_name).stream()
        for v in videos:
            v_data = v.to_dict()
            video_src = v_data.get("videoSrc", "")
            video_name = v_data.get("videoName", "بدون اسم")
            description = v_data.get("description", "")
            # استخراج معرف الفيديو للحصول على الصورة المصغرة
            video_id = ""
            if "embed/" in video_src:
                video_id = video_src.split("embed/")[1].split("?")[0]
            thumbnail = f"https://img.youtube.com/vi/{video_id}/hqdefault.jpg" if video_id else ""
            videos_html += f'''
            <div class="video-item">
              <img src="{thumbnail}" alt="{video_name}" onclick="openVideoModal('{video_src}')">
              <p>{video_name}</p>
              <p>{description}</p>
            </div>
            '''
    except Exception as e:
        videos_html = f"<p>حدث خطأ: {e}</p>"
    return render_template_string(html, folder_name=folder_name, videos_html=videos_html, current_uid=current_uid)












# =====================================================================
# مسار لحفظ بيانات الفيديو في Firestore (مثال)
# =====================================================================
@app.route('/save_video', methods=['POST'])
def save_video():
    if 'user' not in session:
        return jsonify(success=False, error="المستخدم غير مسجل")
    data = request.get_json()
    try:
        video_data = {
            "videoSrc": data.get("videoSrc"),
            "videoName": data.get("videoName"),
            "folder": data.get("folder"),
            "description": data.get("description"),
            "user": session['user']
        }
        # حفظ البيانات في مجموعة "videos" داخل Firestore
        db.collection("videos").add(video_data)
        return jsonify(success=True)
    except Exception as e:
        return jsonify(success=False, error=str(e))
    
    
    
    
    
    
    
    
    
    
    
# =====================================================================
# صفحة دخول الكمبيوتر الخاص بالمستخدم
# =====================================================================
@app.route('/computer/<user_id>', endpoint='computer_page')
def computer(user_id):
    # شاشة كمبيوتر مبسطة للمستخدم (يمكن تعديل التصميم حسب المتطلبات)
    html = """
    <!DOCTYPE html>
    <html lang="ar">
    <head>
      <meta charset="UTF-8">
      <title>كمبيوتر المستخدم</title>
      <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap" rel="stylesheet">
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
      {% raw %}
      <style>
        body {
          font-family: 'Poppins', sans-serif;
          background: #141e30;
          color: #f0f0f0;
          display: flex;
          justify-content: center;
          align-items: center;
          height: 100vh;
          text-align: center;
        }
        button {
          padding: 10px 20px;
          background: #00d1b2;
          border: none;
          border-radius: 5px;
          color: #fff;
          cursor: pointer;
          margin-top: 20px;
        }
        button:hover { background: #008a80; }
      </style>
      {% endraw %}
    </head>
    <body>
      <h1>شاشة الكمبيوتر للمستخدم: {{ user_id }}</h1>
      <p>هنا يتم عرض شاشة الكمبيوتر الخاصة بالمستخدم.</p>
      <button onclick="window.history.back();">عودة</button>
    </body>
    </html>
    """
    return render_template_string(html, user_id=user_id)
  
  
  
  
  
  
  
  
  
  
  

# =====================================================================
# صفحة "مستخدمو الموقع" (بطاقات المستخدمين)
# =====================================================================
@app.route('/users')
def users_page():
    if 'user' not in session:
        return redirect(url_for('welcome'))
    current_uid = session['user']
    user_cards_html = ""
    try:
        users = db.collection("users").stream()
        for u in users:
            u_data = u.to_dict()
            u_uid = u.id
            email = u_data.get("email", "بدون اسم")
            letter = email[0].upper() if email else "U"
            user_cards_html += f'''
            <div class="user-card">
              <div class="user-icon">{letter}</div>
              <div class="user-email">{email}</div>
              <div class="user-actions">
                <a href="mailto:{email}" title="إرسال بريد إلكتروني"><i class="fa-solid fa-envelope"></i></a>
                <a href="/computer/{u_uid}" title="دخول الكمبيوتر الخاص بالمستخدم"><i class="fa-solid fa-desktop"></i></a>
              </div>
            </div>
            '''
    except Exception as e:
        user_cards_html = f"<p>حدث خطأ: {e}</p>"
    html = """
    <!DOCTYPE html>
    <html lang="ar">
    <head>
      <meta charset="UTF-8">
      <title>مستخدمو الموقع</title>
      <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500&display=swap" rel="stylesheet">
      <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
      {% raw %}
      <style>
        body {
          font-family: 'Poppins', sans-serif;
          background: #141e30;
          color: #f0f0f0;
          margin: 0;
          padding: 20px;
        }
        .header {
          text-align: center;
          margin-bottom: 20px;
        }
        .user-cards-container {
          display: flex;
          flex-wrap: wrap;
          gap: 20px;
          justify-content: center;
        }
        .user-card {
          background: rgba(0,0,0,0.6);
          border: 1px solid #00d1b2;
          border-radius: 10px;
          padding: 15px;
          width: 200px;
          text-align: center;
          box-shadow: 0 4px 10px rgba(0,0,0,0.3);
          transition: transform 0.3s;
        }
        .user-card:hover {
          transform: scale(1.05);
        }
        .user-icon {
          font-size: 3em;
          color: #00d1b2;
          margin-bottom: 10px;
        }
        .user-email {
          font-size: 1em;
          margin-bottom: 10px;
        }
        .user-actions a {
          color: #00d1b2;
          margin: 0 5px;
          font-size: 1.5em;
          text-decoration: none;
        }
        .user-actions a:hover {
          color: #f39c12;
        }
        .back-btn {
          display: block;
          margin: 20px auto;
          padding: 10px 20px;
          background: #00d1b2;
          border: none;
          border-radius: 5px;
          color: #fff;
          text-decoration: none;
          text-align: center;
          width: 150px;
        }
        .back-btn:hover {
          background: #008a80;
        }
      </style>
      {% endraw %}
    </head>
    <body>
      <div class="header">
        <h1>مستخدمو الموقع</h1>
      </div>
      <div class="user-cards-container">
        {{ user_cards|safe }}
      </div>
      <a href="/home" class="back-btn">عودة للرئيسية</a>
    </body>
    </html>
    """
    return render_template_string(html, user_cards=user_cards_html)
  
  

# =====================================================================
# صفحة دخول الكمبيوتر الخاص بالمستخدم
# =====================================================================
# (تم تعريف هذا المسار مرة واحدة فقط لتجنب التكرار)
# =====================================================================
@app.route('/logout')
def logout():
    session.pop('user', None)
    return redirect(url_for('welcome'))


# إضافة مسارات جديدة لكل قسم
@app.route('/home/<section>')
def section_page(section):
    if 'user' not in session:
        return redirect(url_for('welcome'))
    current_uid = session['user']
    try:
        user = auth.get_user(current_uid)
        doc_ref = db.collection("users").document(current_uid)
        doc = doc_ref.get()
        if doc.exists:
            user_data = doc.to_dict()
            current_email = user_data.get("email", "غير متوفر")
            current_username = user_data.get("username", "غير متوفر")
            current_avatar = user_data.get("avatar", "")
        else:
            current_email = "غير متوفر"
            current_username = "غير متوفر"
            current_avatar = ""
    except Exception as e:
        current_email = f"خطأ: {e}"
        current_username = "غير متوفر"
        current_avatar = "غير متوفر"

    # تحديث الHTML ليشمل القسم النشط المحدد
    html = """
    <!DOCTYPE html>
    <html lang="ar">
    <head>
        <!-- ...existing head content... -->
    </head>
    <body>
        <div class="container">
            <div class="sidebar">
                <a href="/home/chat" title="الشات" onclick="showSection('chat', event)">
                    <i class="fa-solid fa-comment-dots"></i>
                </a>
                <a href="/home/feed" title="الفيد" onclick="showSection('feed', event)">
                    <i class="fa-solid fa-newspaper"></i>
                </a>
                <a href="/home/profile" title="البروفايل" onclick="showSection('profile', event)">
                    <i class="fa-solid fa-user"></i>
                </a>
                <a href="/home/videos" title="الفيديوهات" onclick="showSection('videos', event)">
                    <i class="fa-solid fa-video"></i>
                </a>
                <a href="/home/computer" class="computer-icon" title="الكمبيوتر" onclick="showSection('computer', event)">
                    <i class="fa-solid fa-desktop"></i>
                </a>
                <a href="/users" class="user-icon" title="مستخدمو الموقع">
                    <i class="fa-solid fa-users"></i>
                </a>
                <div class="bottom-icons">
                    <a href="/home/trash" class="trash-icon" title="سلة المهملات" onclick="showSection('trash', event)">
                        <i class="fa-solid fa-trash"></i>
                    </a>
                    <a href="/logout" class="close-icon" title="إغلاق">
                        <i class="fa-solid fa-xmark"></i>
                    </a>
                </div>
            </div>
            <!-- ...existing content... -->
        </div>
        <script>
            // تحديث دالة showSection
            function showSection(sectionId, event) {
                if (event) {
                    event.preventDefault();
                }
                
                // إخفاء جميع الأقسام
                var sections = document.getElementsByClassName('section');
                for (var i = 0; i < sections.length; i++) {
                    sections[i].classList.remove('active');
                }
                
                // إظهار القسم المحدد
                var selectedSection = document.getElementById(sectionId);
                if (selectedSection) {
                    selectedSection.classList.add('active');
                }
                
                // تحديث الرابط في المتصفح
                var newUrl = '/home/' + sectionId;
                history.pushState({section: sectionId}, '', newUrl);
            }

            // التأكد من إظهار القسم الصحيح عند تحميل الصفحة
            document.addEventListener('DOMContentLoaded', function() {
                var currentSection = '{{ section }}';
                showSection(currentSection);
            });

            // التعامل مع أزرار التنقل في المتصفح
            window.addEventListener('popstate', function(event) {
                if (event.state && event.state.section) {
                    showSection(event.state.section);
                }
            });

            // ...existing JavaScript code...
        </script>
    </body>
    </html>
    """
    return render_template_string(html, uid=current_uid, email=current_email, 
                                username=current_username, avatar=current_avatar,
                                section=section)

if __name__ == '__main__':
    app.run(debug=True)
