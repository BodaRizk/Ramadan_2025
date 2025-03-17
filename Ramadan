from tkinter import *
from tkinter import ttk, messagebox
import requests
import configparser
import os
import datetime
import threading

# استيراد المكتبات مع معالجة الأخطاء
try:
    from playsound import playsound
except ImportError:
    messagebox.showwarning("تنبيه", "لم يتم العثور على مكتبة playsound. الرجاء تثبيتها باستخدام: pip install playsound")
    
# ثوابت التطبيق
SETTINGS_FILE = 'settings.ini'
COLORS = {
    "light": {"bg": "#f0f0f0", "fg": "#333333", "accent": "#4CAF50", "button_bg": "#4CAF50", "button_fg": "white", "highlight": "#8BC34A"},
    "dark": {"bg": "#2C3E50", "fg": "#ECEFF1", "accent": "#16A085", "button_bg": "#16A085", "button_fg": "white", "highlight": "#1ABC9C"}
}

# قائمة بأصوات المؤذنين مع روابط التحميل
adhan_sounds = {
    "عبد الرحمن السديس": "https://download.quranicaudio.com/quran/abdulrahman_al_sudais/adhan.mp3",
    "مشاري بن راشد العفاسي": "https://download.quranicaudio.com/quran/mishari_al_afasy/adhan.mp3",
    "محمد رفعت": "https://www.al-azkar.com/2024/08/azan-mp3.html#الاذان-بصوت-الشيخ-محمد-رفعت",
    "عبد الباسط عبد الصمد": "https://download.quranicaudio.com/quran/abdul_basit_murattal/adhan.mp3",
    "ماهر المعيقلي": "https://download.quranicaudio.com/quran/maher_al_muaiqly/adhan.mp3",
    "سعد الغامدي": "https://download.quranicaudio.com/quran/saad_al_ghamdi/adhan.mp3",
    "خالد الجليل": "https://download.quranicaudio.com/quran/khalid_al_jaleel/adhan.mp3",
    "إدريس أبكر": "https://download.quranicaudio.com/quran/idrees_abkr/adhan.mp3",
}

# مواقيت الصلاة لشهر مارس 2025 - تم تبسيط التنسيق
prayer_times = {
    '01': (4,57,12,5,15,22,17,51,19,21), '02': (4,55,12,5,15,23,17,52,19,22),
    '03': (4,54,12,5,15,23,17,52,19,22), '04': (4,53,12,4,15,23,17,53,19,23),
    '05': (4,52,12,4,15,24,17,54,19,24), '06': (4,51,12,4,15,24,17,55,19,25),
    '07': (4,50,12,4,15,24,17,55,19,25), '08': (4,49,12,3,15,25,17,56,19,26),
    '09': (4,47,12,3,15,25,17,57,19,27), '10': (4,46,12,3,15,25,17,57,19,27),
    '11': (4,45,12,3,15,26,17,58,19,28), '12': (4,44,12,2,15,26,17,59,19,29),
    '13': (4,42,12,2,15,26,18,0,19,30), '14': (4,41,12,2,15,26,18,0,19,30),
    '15': (4,40,12,2,15,27,18,1,19,31), '16': (4,38,12,1,15,27,18,2,19,32),
    '17': (4,37,12,1,15,27,18,2,19,32), '18': (4,36,12,1,15,27,18,3,19,33),
    '19': (4,35,12,0,15,27,18,4,19,34), '20': (4,33,12,0,15,27,18,4,19,34),
    '21': (4,32,12,0,15,28,18,5,19,35), '22': (4,31,12,0,15,28,18,6,19,36),
    '23': (4,29,11,59,15,28,18,6,19,36), '24': (4,28,11,59,15,28,18,7,19,37),
    '25': (4,26,11,59,15,28,18,8,19,38), '26': (4,25,11,58,15,28,18,8,19,38),
    '27': (4,24,11,58,15,28,18,9,19,39), '28': (4,22,11,58,15,29,18,10,19,40),
    '29': (4,21,11,57,15,29,18,10,19,40), '30': (4,20,11,57,15,29,18,11,19,41),
    '31': (4,18,11,57,15,29,18,12,19,42)
}

prayers = ['فجر', 'ظهر', 'عصر', 'مغرب', 'عشاء']
prayer_colors = ['#FFC107', '#FF9800', '#FF5722', '#3F51B5', '#303F9F']
config = configparser.ConfigParser()

class CircularProgressbar(Canvas):
    """رسم شريط تقدم دائري (كعكة مفرغة)"""
    def __init__(self, parent, width=200, height=200, **kwargs):
        super().__init__(parent, width=width, height=height, **kwargs)
        self.width = width
        self.height = height
        self.config(highlightthickness=0, bg=kwargs.get('bg', '#f0f0f0'))
        
    def draw_progress(self, percentage, color, text="", text_color="#333333"):
        self.delete("all")
        
        # اختصار الإعدادات
        padding = 10
        x0, y0 = padding, padding
        x1, y1 = self.width - padding, self.height - padding
        x_center, y_center = self.width // 2, self.height // 2
        
        # رسم الدائرة الخلفية
        self.create_oval(x0, y0, x1, y1, fill="", outline="#E0E0E0", width=10)
        
        # رسم القوس
        angle = 360 * (percentage / 100)
        self.create_arc(x0, y0, x1, y1, start=90, extent=-angle, style="arc", outline=color, width=10)
        
        # إضافة النصوص
        self.create_text(x_center, y_center - 15, text=f"{int(percentage)}%", fill=text_color, font=("Arial", 24, "bold"))
        if text:
            self.create_text(x_center, y_center + 15, text=text, fill=text_color, font=("Arial", 12))

def load_settings():
    """تحميل إعدادات التطبيق"""
    default_settings = {
        'default_sound': list(adhan_sounds.keys())[0],
        'dark_mode': False,
        'notifications': True
    }
    
    if not os.path.exists(SETTINGS_FILE):
        return default_settings
        
    try:
        config.read(SETTINGS_FILE)
        return {
            'default_sound': config.get('Settings', 'default_sound', fallback=default_settings['default_sound']),
            'dark_mode': config.getboolean('Settings', 'dark_mode', fallback=default_settings['dark_mode']),
            'notifications': config.getboolean('Settings', 'notifications', fallback=default_settings['notifications'])
        }
    except Exception as e:
        messagebox.showwarning("تحذير", f"خطأ في قراءة الإعدادات: {e}")
        return default_settings

def save_settings(settings):
    """حفظ إعدادات التطبيق"""
    try:
        config['Settings'] = settings
        with open(SETTINGS_FILE, 'w') as configfile:
            config.write(configfile)
    except Exception as e:
        messagebox.showerror("خطأ", f"خطأ في حفظ الإعدادات: {e}")

def load_sound(sound_name, sound_url):
    """تحميل ملف الصوت"""
    local_file = f"{sound_name}.mp3"
    if not os.path.exists(local_file):
        try:
            response = requests.get(sound_url)
            with open(local_file, "wb") as file:
                file.write(response.content)
        except Exception as e:
            messagebox.showerror("خطأ", f"خطأ في تحميل الصوت: {e}")
            return None
    return local_file

def play_adhan(sound_name, status_label=None):
    """تشغيل صوت الأذان في خيط منفصل"""
    def play_sound_thread():
        if status_label:
            status_label.config(text="جاري تحميل صوت الأذان...")
        
        try:
            local_file = load_sound(sound_name, adhan_sounds[sound_name])
            
            if local_file:
                if status_label:
                    status_label.config(text="جاري تشغيل الأذان...")
                
                playsound(local_file)
                
                if status_label:
                    status_label.config(text="تم تشغيل الأذان بنجاح")
        except Exception as e:
            if status_label:
                status_label.config(text=f"خطأ: {str(e)}")
            messagebox.showerror("خطأ", f"خطأ في تشغيل الأذان: {e}")
    
    threading.Thread(target=play_sound_thread, daemon=True).start()

# تحسين دالة إنشاء كائن التاريخ والوقت
def create_datetime(day, hour, minute):
    try:
        return datetime.datetime(2025, 3, int(day), hour, minute)
    except ValueError:
        return datetime.datetime.now()

# تحسين إنشاء قاموس أوقات الصلاة
prayer_datetimes = {}
for day, times in prayer_times.items():
    prayer_datetimes[day] = {
        prayers[i]: create_datetime(day, times[2*i], times[2*i+1])
        for i in range(len(prayers))
    }

def calculate_times(prayer_datetimes):
    """حساب وقت السحور والإمساك"""
    suhur_times, imsak_times = {}, {}
    
    for day_str, day_prayers in prayer_datetimes.items():
        fajr_time = day_prayers['فجر']
        suhur_times[day_str] = fajr_time - datetime.timedelta(minutes=20)
        imsak_times[day_str] = fajr_time - datetime.timedelta(minutes=140)
        
    return suhur_times, imsak_times

suhur_times, imsak_times = calculate_times(prayer_datetimes)

def get_next_prayer():
    """الحصول على موعد الصلاة التالية"""
    now = datetime.datetime.now()
    day_str = now.strftime('%d')
    
    if day_str not in prayer_datetimes:
        return None, None, None
        
    today_prayers = prayer_datetimes[day_str]
    
    # البحث عن الصلاة التالية
    for prayer in prayers:
        prayer_time = today_prayers[prayer]
        if prayer_time > now:
            return prayer, prayer_time, prayer_time - now
    
    # إذا لم يتم العثور على صلاة تالية، ابحث في اليوم التالي
    tomorrow = (now + datetime.timedelta(days=1)).strftime('%d')
    if tomorrow in prayer_datetimes:
        return 'فجر', prayer_datetimes[tomorrow]['فجر'], prayer_datetimes[tomorrow]['فجر'] - now
    
    return None, None, None

def format_time_diff(time_diff):
    """تنسيق الفرق الزمني"""
    if not time_diff:
        return "غير متوفر"
    
    total_seconds = int(time_diff.total_seconds())
    hours = total_seconds // 3600
    minutes = (total_seconds % 3600) // 60
    seconds = total_seconds % 60
    
    return f"{hours:02d}:{minutes:02d}:{seconds:02d}"

def calculate_percentage(time_diff):
    """حساب النسبة المئوية للوقت المتبقي"""
    if not time_diff:
        return 0
    
    # افتراض أقصى وقت بين صلاتين هو 6 ساعات
    max_seconds = 6 * 60 * 60
    seconds_left = time_diff.total_seconds()
    
    # حدود النسبة المئوية بين 0 و 100
    if seconds_left >= max_seconds:
        return 0
    return (1 - (seconds_left / max_seconds)) * 100

def show_prayer_times(theme):
    """عرض مواقيت الصلاة"""
    colors = COLORS[theme]
    
    times_window = Toplevel()
    times_window.title("مواقيت الصلاة")
    times_window.geometry("550x700")
    times_window.configure(bg=colors["bg"])
    
    # إطار العنوان
    header_frame = Frame(times_window, bg=colors["accent"], padx=10, pady=10)
    header_frame.pack(fill=X)
    Label(header_frame, text="مواقيت الصلاة", font=("Arial", 20, "bold"), 
          fg="white", bg=colors["accent"]).pack(side=RIGHT, padx=10)
    
    # إطار المحتوى
    content_frame = Frame(times_window, bg=colors["bg"], padx=20, pady=20)
    content_frame.pack(fill=BOTH, expand=True)
    
    # عرض التاريخ الحالي
    now = datetime.datetime.now()
    day_str = now.strftime('%d')
    
    Label(content_frame, text=f"التاريخ: {now.strftime('%A %d %B %Y')}", font=("Arial", 16),
          bg=colors["bg"], fg=colors["fg"]).pack(pady=10)
    
    # عرض الصلاة التالية
    next_prayer, next_prayer_time, time_diff = get_next_prayer()
    
    if next_prayer and next_prayer_time:
        next_frame = Frame(content_frame, bg=colors["bg"], padx=10, pady=10)
        next_frame.pack(fill=X, pady=10)
        
        Label(next_frame, text=f"الصلاة التالية: {next_prayer}", font=("Arial", 18, "bold"),
              bg=colors["bg"], fg=colors["accent"]).pack(pady=5)
        
        Label(next_frame, text=f"الوقت: {next_prayer_time.strftime('%I:%M %p')}", font=("Arial", 16),
              bg=colors["bg"], fg=colors["fg"]).pack(pady=5)
        
        Label(next_frame, text=f"المتبقي: {format_time_diff(time_diff)}", font=("Arial", 16),
              bg=colors["bg"], fg=colors["fg"]).pack(pady=5)
        
        # رسم الكعكة المفرغة
        progress = calculate_percentage(time_diff)
        prayer_index = prayers.index(next_prayer) if next_prayer in prayers else 0
        progress_color = prayer_colors[prayer_index]
        
        progress_canvas = CircularProgressbar(next_frame, width=150, height=150, bg=colors["bg"])
        progress_canvas.pack(pady=10)
        progress_canvas.draw_progress(progress, progress_color, text="متبقي للصلاة", text_color=colors["fg"])
    
    # جدول مواقيت الصلاة
    if day_str in prayer_datetimes:
        table_frame = Frame(content_frame, bg=colors["bg"], padx=10, pady=10)
        table_frame.pack(fill=X, pady=10)
        
        Label(table_frame, text="مواقيت الصلاة لليوم", font=("Arial", 18, "bold"),
              bg=colors["bg"], fg=colors["accent"]).pack(pady=10)
        
        # جدول الصلوات
        prayer_table = ttk.Treeview(table_frame, columns=("prayer", "time"), show="headings")
        prayer_table.heading("prayer", text="الصلاة")
        prayer_table.heading("time", text="الوقت")
        prayer_table.column("prayer", width=150, anchor=CENTER)
        prayer_table.column("time", width=150, anchor=CENTER)
        
        # إضافة البيانات
        today_prayers = prayer_datetimes[day_str]
        for i, prayer in enumerate(prayers):
            time_str = today_prayers[prayer].strftime("%I:%M %p")
            prayer_table.insert("", END, values=(prayer, time_str), tags=(f"prayer{i}",))
            prayer_table.tag_configure(f"prayer{i}", background=prayer_colors[i], foreground="white")
        
        prayer_table.pack(pady=10)
        
        # عرض السحور والإمساك
        ramadan_frame = Frame(content_frame, bg=colors["bg"], padx=10, pady=10)
        ramadan_frame.pack(fill=X, pady=10)
        
        Label(ramadan_frame, text="السحور والإمساك", font=("Arial", 18, "bold"),
              bg=colors["bg"], fg=colors["accent"]).pack(pady=10)
        
        suhur_time_str = suhur_times[day_str].strftime("%I:%M %p")
        imsak_time_str = imsak_times[day_str].strftime("%I:%M %p")
        
        # إطار السحور
        suhur_frame = Frame(ramadan_frame, bg=colors["bg"])
        suhur_frame.pack(fill=X, pady=5)
        Label(suhur_frame, text="السحور:", width=10, font=("Arial", 14),
              bg=colors["bg"], fg=colors["fg"]).pack(side=RIGHT)
        Label(suhur_frame, text=suhur_time_str, font=("Arial", 14, "bold"),
              bg=colors["bg"], fg=colors["highlight"]).pack(side=RIGHT, padx=10)
        
        # إطار الإمساك
        imsak_frame = Frame(ramadan_frame, bg=colors["bg"])
        imsak_frame.pack(fill=X, pady=5)
        Label(imsak_frame, text="الإمساك:", width=10, font=("Arial", 14),
              bg=colors["bg"], fg=colors["fg"]).pack(side=RIGHT)
        Label(imsak_frame, text=imsak_time_str, font=("Arial", 14, "bold"),
              bg=colors["bg"], fg=colors["highlight"]).pack(side=RIGHT, padx=10)
    else:
        Label(content_frame, text="لا توجد بيانات لليوم الحالي", font=("Arial", 14),
              bg=colors["bg"], fg=colors["fg"]).pack(pady=5)
    
    # زر الإغلاق
    Button(times_window, text="إغلاق", font=("Arial", 14),
           bg=colors["button_bg"], fg=colors["button_fg"], padx=20,
           command=times_window.destroy).pack(pady=20)

def open_advanced_settings(parent, current_settings, on_save):
    """فتح نافذة الإعدادات المتقدمة"""
    theme = "dark" if current_settings.get('dark_mode', False) else "light"
    colors = COLORS[theme]
    
    settings_window = Toplevel(parent)
    settings_window.title("التخصيص المتقدم")
    settings_window.geometry("450x500")
    settings_window.configure(bg=colors["bg"])
    
    # عنوان النافذة
    Label(settings_window, text="تخصيص البرنامج", font=("Arial", 18, "bold"),
          bg=colors["bg"], fg=colors["accent"]).pack(pady=20)
    
    # إطار الإعدادات
    settings_frame = Frame(settings_window, padx=20, pady=10, bg=colors["bg"])
    settings_frame.pack(fill=X)
    
    # اختيار صوت المؤذن
    Label(settings_frame, text="اختر صوت المؤذن:", font=("Arial", 14),
          bg=colors["bg"], fg=colors["fg"]).pack(pady=5, anchor=W)
    
    selected_sound = StringVar(settings_window)
    selected_sound.set(current_settings.get('default_sound', list(adhan_sounds.keys())[0]))
    
    ttk.Combobox(settings_frame, textvariable=selected_sound,
                 values=list(adhan_sounds.keys()), state="readonly",
                 font=("Arial", 12), width=30).pack(pady=10, fill=X)
    
    # تفعيل الوضع الداكن
    dark_mode_var = BooleanVar(value=current_settings.get('dark_mode', False))
    ttk.Checkbutton(settings_frame, text="تفعيل الوضع الداكن",
                   variable=dark_mode_var).pack(fill=X, pady=10)
    
    # تفعيل إشعارات الصلاة
    notifications_var = BooleanVar(value=current_settings.get('notifications', True))
    ttk.Checkbutton(settings_frame, text="تفعيل إشعارات الصلاة",
                   variable=notifications_var).pack(fill=X, pady=10)
    
    # اختبار صوت الأذان
    test_frame = Frame(settings_frame, bg=colors["bg"])
    test_frame.pack(fill=X, pady=20)
    
    status_label = Label(test_frame, text="", font=("Arial", 10),
                        bg=colors["bg"], fg=colors["fg"])
    status_label.pack(pady=5)
    
    Button(test_frame, text="اختبار صوت الأذان", font=("Arial", 12),
           bg=colors["button_bg"], fg=colors["button_fg"],
           command=lambda: play_adhan(selected_sound.get(), status_label)).pack(pady=5)
    
    # أزرار الإجراءات
    buttons_frame = Frame(settings_window, padx=20, pady=20, bg=colors["bg"])
    buttons_frame.pack(fill=X, side=BOTTOM)
    
    # زر الإلغاء
    Button(buttons_frame, text="إلغاء", font=("Arial", 12), bg="#F44336", fg="white", padx=20,
           command=settings_window.destroy).pack(side=LEFT, padx=10)
    
    # زر الحفظ
    Button(buttons_frame, text="حفظ التخصيصات", font=("Arial", 12),
           bg=colors["button_bg"], fg=colors["button_fg"], padx=20,
           command=lambda: [
               on_save({
                   'default_sound': selected_sound.get(),
                   'dark_mode': dark_mode_var.get(),
                   'notifications': notifications_var.get()
               }),
               settings_window.destroy()
           ]).pack(side=RIGHT, padx=10)

def apply_settings(root, settings):
    """تطبيق الإعدادات على واجهة المستخدم"""
    theme = "dark" if settings.get('dark_mode', False) else "light"
    colors = COLORS[theme]
    
    root.configure(bg=colors["bg"])
    
    for widget in root.winfo_children():
        if isinstance(widget, (Frame, Label)):
            widget.configure(bg=colors["bg"])
            
        if isinstance(widget, Label):
            widget.configure(fg=colors["fg"])
            
        if isinstance(widget, Button):
            widget.configure(bg=colors["button_bg"], fg=colors["button_fg"])

def main():
    """الدالة الرئيسية للتطبيق"""
    root = Tk()
    root.title("تطبيق الأذان")
    
    # تحميل الإعدادات
    settings = load_settings()
    theme = "dark" if settings.get('dark_mode', False) else "light"
    colors = COLORS[theme]
    
    # متغيرات الواجهة
    selected_sound = StringVar(root)
    selected_sound.set(settings.get('default_sound', list(adhan_sounds.keys())[0]))
    
    # تصميم الواجهة
    root.configure(bg=colors["bg"])
    root.geometry("400x500")
    
    # عنوان التطبيق
    title_frame = Frame(root, bg=colors["bg"], pady=20)
    title_frame.pack(fill=X)
    
    title_label = Label(title_frame, text="تطبيق الأذان", font=("Arial", 22, "bold"),
                       bg=colors["bg"], fg=colors["accent"])
    title_label.pack()
    
    # اختيار صوت المؤذن
    sound_frame = Frame(root, bg=colors["bg"], pady=10)
    sound_frame.pack(fill=X, padx=20)
    
    sound_label = Label(sound_frame, text="اختر صوت المؤذن:", font=("Arial", 14),
                       bg=colors["bg"], fg=colors["fg"])
    sound_label.pack(pady=5)
    
    sound_combo = ttk.Combobox(sound_frame, textvariable=selected_sound,
                             values=list(adhan_sounds.keys()), state="readonly",
                             font=("Arial", 12), width=30)
    sound_combo.pack(pady=5)
    
    # زر تشغيل الأذان
    play_frame = Frame(root, bg=colors["bg"], pady=10)
    play_frame.pack(fill=X)
    
    play_button = Button(play_frame, text="تشغيل الأذان", font=("Arial", 16, "bold"),
                        bg=colors["button_bg"], fg=colors["button_fg"], padx=20, pady=10,
                        command=lambda: play_adhan(selected_sound.get()))
    play_button.pack(pady=10)
    
    # زر عرض مواقيت الصلاة
    times_button = Button(root, text="عرض مواقيت الصلاة", font=("Arial", 14),
                         bg=colors["button_bg"], fg=colors["button_fg"], padx=20, pady=5,
                         command=lambda: show_prayer_times(theme))
    times_button.pack(pady=15)
    
    # عرض الصلاة التالية
    next_frame = Frame(root, bg=colors["bg"], pady=10, padx=20)
    next_frame.pack(fill=X)
    
    next_prayer, next_time, time_diff = get_next_prayer()
    if next_prayer and next_time:
        Label(next_frame, text="الصلاة التالية:", font=("Arial", 14),
              bg=colors["bg"], fg=colors["fg"]).pack(anchor=W, pady=2)
        
        Label(next_frame, text=f"{next_prayer} - {next_time.strftime('%I:%M %p')}", font=("Arial", 16, "bold"),
              bg=colors["bg"], fg=colors["accent"]).pack(anchor=W, pady=2)
        
        Label(next_frame, text=f"المتبقي: {format_time_diff(time_diff)}", font=("Arial", 14),
              bg=colors["bg"], fg=colors["fg"]).pack(anchor=W, pady=2)
    
    # زر التخصيص المتقدم
    settings_button = Button(root, text="التخصيص المتقدم", font=("Arial", 14),
                            bg=colors["button_bg"], fg=colors["button_fg"], padx=20, pady=5,
                            command=lambda: open_advanced_settings(root, settings, 
                                          lambda new_settings: [
                                              save_settings(new_settings),
                                              apply_settings(root, new_settings)
                                          ]))
    settings_button.pack(pady=15)
    
    # زر الإغلاق
    close_button = Button(root, text="إغلاق", font=("Arial", 14),
                         bg="#F44336", fg="white", padx=20, pady=5,
                         command=lambda: [                         save_settings(settings), root.destroy()])
    close_button.pack(pady=15)

    root.protocol("WM_DELETE_WINDOW", lambda: [save_settings(settings), root.destroy()])
    root.mainloop()

if __name__ == "__main__":
    main()
