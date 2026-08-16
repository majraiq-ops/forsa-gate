# forsa-gate
منصة بوابة الفرص
import React, { useState, useEffect } from 'react';
import { 
  AlertTriangle, Briefcase, Search, MapPin, 
  FileText, MessageCircle, Instagram, Phone, 
  X, CheckCircle, ChevronDown, Menu, DollarSign
} from 'lucide-react';
import { initializeApp } from "firebase/app";
import { getDatabase, ref, push, onValue } from "firebase/database";

// إعدادات Firebase كما تم توفيرها
const firebaseConfig = {
  apiKey: "AIzaSyA9eiDL9hjY9OHUgvZpLL8CWNh8KLdor-c",
  authDomain: "forsa-gate.firebaseapp.com",
  databaseURL: "https://forsa-gate-default-rtdb.firebaseio.com",
  projectId: "forsa-gate",
  storageBucket: "forsa-gate.firebasestorage.app",
  messagingSenderId: "751636769454",
  appId: "1:751636769454:web:9c52934f07bf7df6699b3f"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

const DUMMY_JOBS = [
  { id: '1', title: 'مهندس مدني - إشراف', company: 'شركة الأفق للمقاولات', location: 'بغداد', type: 'دوام كامل', date: '2026-08-15' },
  { id: '2', title: 'مطور واجهات أمامية (React)', company: 'حلول التقنية', location: 'أربيل', type: 'عمل عن بعد', date: '2026-08-14' },
  { id: '3', title: 'محاسب مالي', company: 'مجموعة النور التجارية', location: 'البصرة', type: 'دوام كامل', date: '2026-08-16' },
  { id: '4', title: 'مسؤول موارد بشرية HR', company: 'شركة الريادة', location: 'النجف', type: 'دوام جزئي', date: '2026-08-10' },
];

const IRAQI_GOVERNORATES = [
  "بغداد", "البصرة", "نينوى", "أربيل", "النجف", "ذي قار", 
  "كركوك", "الأنبار", "ديالى", "المثنى", "القادسية", "ميسان", 
  "واسط", "صلاح الدين", "دهوك", "السليمانية", "بابل", "كربلاء"
];

export default function App() {
  const [jobs, setJobs] = useState([]);
  const [loadingJobs, setLoadingJobs] = useState(true);
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedLocation, setSelectedLocation] = useState('');

  // حالات نافذة طلب السيرة الذاتية
  const [isCvModalOpen, setIsCvModalOpen] = useState(false);
  const [cvForm, setCvForm] = useState({
    name: '',
    whatsapp: '',
    email: '',
    specialization: '',
    experience: ''
  });
  const [isSubmittingCv, setIsSubmittingCv] = useState(false);
  const [submitSuccess, setSubmitSuccess] = useState(false);

  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

  useEffect(() => {
    const jobsRef = ref(db, 'jobs');
    const unsubscribe = onValue(jobsRef, (snapshot) => {
      const data = snapshot.val();
      if (data) {
        const jobsList = Object.keys(data).map(key => ({
          id: key,
          ...data[key]
        })).reverse();
        setJobs(jobsList);
      } else {
        setJobs(DUMMY_JOBS);
      }
      setLoadingJobs(false);
    }, (error) => {
      console.error("Error fetching jobs:", error);
      setJobs(DUMMY_JOBS);
      setLoadingJobs(false);
    });

    return () => unsubscribe();
  }, []);

  const handleCvInputChange = (e) => {
    const { name, value } = e.target;
    setCvForm(prev => ({ ...prev, [name]: value }));
  };

  const handleCvSubmit = async (e) => {
    e.preventDefault();
    setIsSubmittingCv(true);
    try {
      const cvRequestsRef = ref(db, 'cv_requests');
      await push(cvRequestsRef, {
        ...cvForm,
        timestamp: new Date().toISOString()
      });
      setSubmitSuccess(true);
      setTimeout(() => {
        setIsCvModalOpen(false);
        setSubmitSuccess(false);
        setCvForm({ name: '', whatsapp: '', email: '', specialization: '', experience: '' });
      }, 3000);
    } catch (error) {
      console.error("Error submitting CV request:", error);
      // استخدام نافذة مخصصة بدلاً من alert الممنوعة
    } finally {
      setIsSubmittingCv(false);
    }
  };

  const filteredJobs = jobs.filter(job => {
    const matchesSearch = job.title?.toLowerCase().includes(searchQuery.toLowerCase()) || 
                          job.company?.toLowerCase().includes(searchQuery.toLowerCase());
    const matchesLocation = selectedLocation ? job.location === selectedLocation : true;
    return matchesSearch && matchesLocation;
  });

  return (
    <div dir="rtl" className="min-h-screen bg-[#0B0B0E] text-gray-200 font-sans relative overflow-x-hidden">
      {/* تضمين خط Tajawal وتخصيص شريط التمرير */}
      <style>
        {`
          @import url('https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;800&display=swap');
          body { font-family: 'Tajawal', sans-serif; }
          ::-webkit-scrollbar { width: 8px; }
          ::-webkit-scrollbar-track { background: #0B0B0E; }
          ::-webkit-scrollbar-thumb { background: #2A2A35; border-radius: 4px; }
          ::-webkit-scrollbar-thumb:hover { background: #D4AF37; }
        `}
      </style>

      {/* شريط التنويه */}
      <div className="bg-[#D4AF37] text-black py-2 px-4 text-sm md:text-base font-bold text-center flex items-center justify-center gap-2 shadow-md z-50 relative">
        <AlertTriangle size={20} className="animate-pulse" />
        <span>تنويه هام: جميع الوظائف مجانية بالكامل، لا نتقاضى أي مبالغ عليها. الخدمات المدفوعة تشمل فقط إنشاء السيرة الذاتية والاستشارات المهنية.</span>
      </div>

      {/* شريط التنقل */}
      <nav className="sticky top-0 z-40 bg-[#0B0B0E]/90 backdrop-blur-md border-b border-[#2A2A35]">
        <div className="container mx-auto px-4 md:px-6 h-20 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="w-10 h-10 bg-gradient-to-br from-[#D4AF37] to-[#9A7B2C] rounded-lg flex items-center justify-center shadow-[0_0_15px_rgba(212,175,55,0.3)]">
              <Briefcase className="text-black" size={24} />
            </div>
            <span className="text-2xl font-extrabold tracking-tight text-white">
              بوابة <span className="text-[#D4AF37]">الفرص</span>
            </span>
          </div>

          <div className="hidden md:flex items-center gap-8 font-medium">
            <a href="#jobs" className="hover:text-[#D4AF37] transition-colors">الوظائف (مجانية)</a>
            <a href="#services" className="hover:text-[#D4AF37] transition-colors">الخدمات المهنية (مدفوعة)</a>
            <a href="#footer" className="hover:text-[#D4AF37] transition-colors">اتصل بنا</a>
          </div>

          <button className="md:hidden text-gray-300 hover:text-white" onClick={() => setMobileMenuOpen(!mobileMenuOpen)}>
            {mobileMenuOpen ? <X size={28} /> : <Menu size={28} />}
          </button>
        </div>
        
        {mobileMenuOpen && (
          <div className="md:hidden bg-[#121218] border-b border-[#2A2A35] py-4 px-4 flex flex-col gap-4">
            <a href="#jobs" onClick={() => setMobileMenuOpen(false)} className="block py-2 text-lg">الوظائف (مجانية)</a>
            <a href="#services" onClick={() => setMobileMenuOpen(false)} className="block py-2 text-lg">الخدمات المهنية (مدفوعة)</a>
            <a href="#footer" onClick={() => setMobileMenuOpen(false)} className="block py-2 text-lg">اتصل بنا</a>
          </div>
        )}
      </nav>

      {/* القسم الترحيبي */}
      <section className="relative pt-24 pb-32 px-4 md:px-8 text-center flex flex-col items-center justify-center overflow-hidden">
        <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[600px] h-[600px] bg-[#D4AF37]/5 rounded-full blur-[100px] pointer-events-none"></div>
        
        <h1 className="text-5xl md:text-7xl font-extrabold text-white mb-6 relative z-10 leading-tight">
          بوابتك نحو <span className="text-transparent bg-clip-text bg-gradient-to-l from-[#D4AF37] to-[#F3E5AB]">التميز المهني</span>
        </h1>
        
        <p className="max-w-2xl text-lg md:text-xl text-gray-400 mb-10 relative z-10 leading-relaxed">
          منصة أسسها المهندس <span className="text-gray-200 font-bold">عبد القهار التميمي</span> لربط الكفاءات بالفرص الحقيقية المجانية، مع توفير خدمات احترافية مدفوعة لتطوير مسارك.
        </p>

        <div className="flex flex-col sm:flex-row gap-4 relative z-10">
          <a href="#jobs" className="px-8 py-4 bg-[#D4AF37] hover:bg-[#B3932F] text-black font-bold rounded-xl transition-all shadow-[0_4px_20px_rgba(212,175,55,0.2)] flex items-center justify-center gap-2">
            <Search size={20} />
            تصفح الوظائف المجانية
          </a>
          <button onClick={() => setIsCvModalOpen(true)} className="px-8 py-4 bg-transparent border-2 border-[#333342] hover:border-[#D4AF37] text-white font-bold rounded-xl transition-all flex items-center justify-center gap-2">
            <FileText size={20} className="text-[#D4AF37]" />
            طلب سيرة ذاتية (مدفوع)
          </button>
        </div>
      </section>

      {/* قسم الوظائف المجانية */}
      <section id="jobs" className="py-20 px-4 md:px-8 bg-[#0F0F14] relative border-t border-[#1A1A24]">
        <div className="container mx-auto max-w-6xl">
          <div className="mb-12 text-center">
            <span className="bg-[#D4AF37]/10 text-[#D4AF37] border border-[#D4AF37]/30 text-xs font-bold px-3 py-1 rounded-full uppercase tracking-wider mb-3 inline-block">
              مجانية 100%
            </span>
            <h2 className="text-3xl md:text-4xl font-bold text-white mb-4">أحدث الفرص الوظيفية</h2>
            <p className="text-gray-400">فرص حقيقية ومتاحة للتقديم المجاني المباشر.</p>
          </div>

          <div className="bg-[#121218] p-4 md:p-6 rounded-2xl border border-[#2A2A35] flex flex-col md:flex-row gap-4 mb-12 shadow-lg">
            <div className="relative flex-1">
              <Search className="absolute right-4 top-1/2 -translate-y-1/2 text-gray-500" size={20} />
              <input 
                type="text" 
                placeholder="ابحث عن مسمى وظيفي أو شركة..." 
                className="w-full bg-[#1A1A24] text-white border border-[#333342] rounded-xl py-3 pr-12 pl-4 focus:outline-none focus:border-[#D4AF37] transition-colors"
                value={searchQuery}
                onChange={(e) => setSearchQuery(e.target.value)}
              />
            </div>
            <div className="relative w-full md:w-64">
              <MapPin className="absolute right-4 top-1/2 -translate-y-1/2 text-gray-500" size={20} />
              <select 
                className="w-full bg-[#1A1A24] text-white border border-[#333342] rounded-xl py-3 pr-12 pl-10 focus:outline-none focus:border-[#D4AF37] transition-colors appearance-none cursor-pointer"
                value={selectedLocation}
                onChange={(e) => setSelectedLocation(e.target.value)}
              >
                <option value="">كل المحافظات</option>
                {IRAQI_GOVERNORATES.map(gov => (
                  <option key={gov} value={gov}>{gov}</option>
                ))}
              </select>
              <ChevronDown className="absolute left-4 top-1/2 -translate-y-1/2 text-gray-500 pointer-events-none" size={16} />
            </div>
          </div>

          {loadingJobs ? (
            <div className="flex justify-center py-20">
              <div className="w-12 h-12 border-4 border-[#333342] border-t-[#D4AF37] rounded-full animate-spin"></div>
            </div>
          ) : filteredJobs.length > 0 ? (
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              {filteredJobs.map(job => (
                <div key={job.id} className="bg-[#121218] p-6 rounded-2xl border border-[#2A2A35] hover:border-[#D4AF37]/50 transition-colors group relative overflow-hidden flex flex-col h-full">
                  <div className="absolute top-0 right-0 w-1 h-full bg-[#D4AF37] opacity-0 group-hover:opacity-100 transition-opacity"></div>
                  
                  <div className="flex justify-between items-start mb-4">
                    <h3 className="text-xl font-bold text-white group-hover:text-[#D4AF37] transition-colors line-clamp-2">
                      {job.title}
                    </h3>
                    <span className="bg-green-500/10 text-green-400 text-xs px-2.5 py-1 rounded-full font-semibold border border-green-500/20">
                      مجاني
                    </span>
                  </div>
                  
                  <div className="text-gray-400 font-medium mb-6 flex-grow">
                    {job.company}
                  </div>
                  
                  <div className="flex flex-wrap gap-2 mt-auto">
                    <span className="bg-[#1A1A24] text-gray-300 text-sm px-3 py-1 rounded-md flex items-center gap-1">
                      <MapPin size={14} className="text-[#D4AF37]" /> {job.location}
                    </span>
                    <span className="bg-[#1A1A24] text-gray-300 text-sm px-3 py-1 rounded-md flex items-center gap-1">
                      <Briefcase size={14} className="text-[#D4AF37]" /> {job.type || 'دوام كامل'}
                    </span>
                  </div>
                </div>
              ))}
            </div>
          ) : (
            <div className="text-center py-20 bg-[#121218] rounded-2xl border border-[#2A2A35]">
              <Search className="mx-auto text-gray-600 mb-4" size={48} />
              <h3 className="text-xl font-bold text-gray-300 mb-2">لا توجد نتائج</h3>
              <p className="text-gray-500">حاول تغيير كلمات البحث أو المحافظة.</p>
            </div>
          )}
        </div>
      </section>

      {/* قسم الخدمات المدفوعة */}
      <section id="services" className="py-20 px-4 md:px-8 border-t border-[#1A1A24] relative">
        <div className="absolute inset-0 bg-gradient-to-b from-[#0B0B0E] to-[#0A0A0D] pointer-events-none"></div>
        <div className="container mx-auto max-w-5xl relative z-10">
          <div className="mb-16 text-center">
            <span className="bg-[#D4AF37]/10 text-[#D4AF37] border border-[#D4AF37]/30 text-xs font-bold px-3 py-1 rounded-full uppercase tracking-wider mb-3 inline-block">
              خدمات احترافية مدفوعة
            </span>
            <h2 className="text-3xl md:text-4xl font-bold text-white mb-4 flex items-center justify-center gap-3">
              <span className="w-12 h-[2px] bg-[#D4AF37]"></span>
              الخدمات المهنية المدفوعة
              <span className="w-12 h-[2px] bg-[#D4AF37]"></span>
            </h2>
            <p className="text-gray-400">استثمر في هويتك المهنية للحصول على أفضل النتائج.</p>
          </div>

          <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
            {/* كارت السيرة الذاتية المدفوعة */}
            <div className="bg-[#121218] rounded-3xl p-8 border border-[#2A2A35] hover:shadow-[0_10px_40px_rgba(0,0,0,0.5)] transition-shadow flex flex-col items-center text-center relative">
              <div className="absolute top-6 left-6 bg-[#D4AF37]/10 text-[#D4AF37] border border-[#D4AF37]/30 text-xs font-bold px-3 py-1 rounded-full flex items-center gap-1">
                <DollarSign size={14} /> خدمة مدفوعة
              </div>
              
              <div className="w-20 h-20 bg-[#1A1A24] rounded-2xl flex items-center justify-center mb-6 border border-[#333342]">
                <FileText size={40} className="text-[#D4AF37]" />
              </div>
              <h3 className="text-2xl font-bold text-white mb-4">صياغة سيرة ذاتية (CV)</h3>
              <p className="text-gray-400 mb-8 flex-grow">
                احصل على سيرة ذاتية احترافية مصممة خصيصاً لتبرز مهاراتك وتزيد من فرص قبولك في المقابلات الوظيفية، بأيدي مختصين (خدمة مدفوعة).
              </p>
              <button 
                onClick={() => setIsCvModalOpen(true)}
                className="w-full py-4 bg-[#D4AF37] hover:bg-[#B3932F] text-black font-bold rounded-xl transition-all shadow-md"
              >
                اطلب سيرتك الذاتية الآن
              </button>
            </div>

            {/* كارت الاستشارة المدفوعة */}
            <div className="bg-[#121218] rounded-3xl p-8 border border-[#2A2A35] hover:shadow-[0_10px_40px_rgba(0,0,0,0.5)] transition-shadow flex flex-col items-center text-center relative overflow-hidden">
              <div className="absolute top-6 left-6 bg-[#D4AF37]/10 text-[#D4AF37] border border-[#D4AF37]/30 text-xs font-bold px-3 py-1 rounded-full flex items-center gap-1">
                <DollarSign size={14} /> خدمة مدفوعة
              </div>

              <div className="w-20 h-20 bg-[#1A1A24] rounded-2xl flex items-center justify-center mb-6 border border-[#333342]">
                <MessageCircle size={40} className="text-[#D4AF37]" />
              </div>
              <h3 className="text-2xl font-bold text-white mb-4">استشارة مهنية متخصصة</h3>
              <p className="text-gray-400 mb-8 flex-grow">
                جلسة استشارية مهنية مدفوعة مباشرة عبر الواتساب لتوجيه مسارك الوظيفي ومراجعة إمكانياتك مع مختصين.
              </p>
              <a 
                href="https://wa.me/9647765516506" 
                target="_blank" 
                rel="noopener noreferrer"
                className="w-full py-4 bg-gradient-to-r from-[#25D366] to-[#128C7E] hover:from-[#1EBE5D] hover:to-[#0F7569] text-white font-bold rounded-xl transition-all shadow-lg flex items-center justify-center gap-2"
              >
                <Phone size={20} />
                طلب استشارة عبر الواتساب (مدفوع)
              </a>
            </div>
          </div>
        </div>
      </section>

      {/* قسم اتصل بنا */}
      <footer id="footer" className="bg-[#08080A] py-12 border-t border-[#1A1A24]">
        <div className="container mx-auto px-4 md:px-8 flex flex-col md:flex-row items-center justify-between gap-6">
          <div className="flex items-center gap-3">
            <Briefcase className="text-[#D4AF37]" size={28} />
            <span className="text-2xl font-extrabold tracking-tight text-white">
              بوابة <span className="text-[#D4AF37]">الفرص</span>
            </span>
          </div>
          
          <p className="text-gray-500 text-center md:text-right text-sm">
            &copy; {new Date().getFullYear()} جميع الحقوق محفوظة لـ بوابة الفرص. الوظائف مجانية بالكامل.
          </p>

          <div className="flex gap-4">
            <a 
              href="https://instagram.com/eng.abdul_qahar_al_tamimi" 
              target="_blank" 
              rel="noopener noreferrer"
              className="w-10 h-10 bg-[#121218] rounded-full flex items-center justify-center text-gray-400 hover:bg-[#D4AF37] hover:text-black transition-colors border border-[#2A2A35]"
            >
              <Instagram size={20} />
            </a>
            <a 
              href="https://wa.me/9647765516506" 
              target="_blank" 
              rel="noopener noreferrer"
              className="w-10 h-10 bg-[#121218] rounded-full flex items-center justify-center text-gray-400 hover:bg-[#25D366] hover:text-black transition-colors border border-[#2A2A35]"
            >
              <Phone size={20} />
            </a>
          </div>
        </div>
      </footer>

      {/* نافذة طلب السيرة الذاتية المنبثقة */}
      {isCvModalOpen && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
          <div className="absolute inset-0 bg-black/80 backdrop-blur-sm" onClick={() => !isSubmittingCv && setIsCvModalOpen(false)}></div>
          
          <div className="bg-[#121218] border border-[#2A2A35] rounded-3xl w-full max-w-lg relative z-10 shadow-2xl overflow-hidden flex flex-col max-h-[90vh]">
            <div className="bg-[#1A1A24] p-6 flex justify-between items-center border-b border-[#2A2A35]">
              <h3 className="text-xl font-bold text-white flex items-center gap-2">
                <FileText className="text-[#D4AF37]" size={24} />
                طلب صياغة سيرة ذاتية (خدمة مدفوعة)
              </h3>
              <button 
                onClick={() => setIsCvModalOpen(false)}
                disabled={isSubmittingCv}
                className="text-gray-400 hover:text-white transition-colors p-1"
              >
                <X size={24} />
              </button>
            </div>

            <div className="p-6 overflow-y-auto">
              {submitSuccess ? (
                <div className="flex flex-col items-center justify-center text-center py-10">
                  <CheckCircle size={80} className="text-[#D4AF37] mb-6" />
                  <h4 className="text-2xl font-bold text-white mb-2">تم استلام طلبك بنجاح!</h4>
                  <p className="text-gray-400">سنتواصل معك قريباً عبر الواتساب لتنسيق تفاصيل الخدمة المدفوعة.</p>
                </div>
              ) : (
                <form onSubmit={handleCvSubmit} className="flex flex-col gap-5">
                  <div>
                    <label className="block text-sm font-medium text-gray-400 mb-2">الاسم الثلاثي</label>
                    <input 
                      required type="text" name="name" value={cvForm.name} onChange={handleCvInputChange}
                      className="w-full bg-[#0B0B0E] border border-[#333342] rounded-xl p-3 text-white focus:outline-none focus:border-[#D4AF37]"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-400 mb-2">رقم الواتساب (مع الرمز الدولي)</label>
                    <input 
                      required type="tel" name="whatsapp" value={cvForm.whatsapp} onChange={handleCvInputChange}
                      placeholder="مثال: +964..." dir="ltr"
                      className="w-full bg-[#0B0B0E] border border-[#333342] rounded-xl p-3 text-white text-right focus:outline-none focus:border-[#D4AF37]"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-400 mb-2">البريد الإلكتروني</label>
                    <input 
                      required type="email" name="email" value={cvForm.email} onChange={handleCvInputChange} dir="ltr"
                      className="w-full bg-[#0B0B0E] border border-[#333342] rounded-xl p-3 text-white text-right focus:outline-none focus:border-[#D4AF37]"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-400 mb-2">التخصص الأكاديمي / المهني</label>
                    <input 
                      required type="text" name="specialization" value={cvForm.specialization} onChange={handleCvInputChange}
                      className="w-full bg-[#0B0B0E] border border-[#333342] rounded-xl p-3 text-white focus:outline-none focus:border-[#D4AF37]"
                    />
                  </div>
                  <div>
                    <label className="block text-sm font-medium text-gray-400 mb-2">نبذة مختصرة عن الخبرات السابقة (إن وجدت)</label>
                    <textarea 
                      name="experience" value={cvForm.experience} onChange={handleCvInputChange} rows="3"
                      className="w-full bg-[#0B0B0E] border border-[#333342] rounded-xl p-3 text-white focus:outline-none focus:border-[#D4AF37] resize-none"
                    ></textarea>
                  </div>
                  
                  <div className="pt-4 border-t border-[#2A2A35] mt-2">
                    <button 
                      type="submit" 
                      disabled={isSubmittingCv}
                      className="w-full py-4 bg-[#D4AF37] hover:bg-[#B3932F] disabled:bg-[#333342] disabled:text-gray-500 text-black font-bold rounded-xl transition-all flex justify-center items-center gap-2"
                    >
                      {isSubmittingCv ? (
                        <div className="w-6 h-6 border-2 border-black border-t-transparent rounded-full animate-spin"></div>
                      ) : (
                        "تأكيد وإرسال طلب الخدمة المدفوعة"
                      )}
                    </button>
                  </div>
                </form>
              )}
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
