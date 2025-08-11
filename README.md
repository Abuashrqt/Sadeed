
import React, { useState, useEffect } from 'react';

// Firebase imports
import { initializeApp } from 'firebase/app';
import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, onAuthStateChanged, signInWithCustomToken, signInAnonymously } from 'firebase/auth';
import { getFirestore, doc, collection, addDoc, onSnapshot, query, orderBy } from 'firebase/firestore';

// Global variables from the canvas environment
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

// Firebase initialization and auth setup
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

// Context for managing global state across components
const AuthContext = React.createContext();

const translations = {
  ar: {
    lang: 'ar',
    dir: 'rtl',
    companyName: 'سديد',
    title: 'سديد للصيانة',
    navHome: 'الرئيسية',
    navServices: 'خدماتنا',
    navAbout: 'من نحن',
    navContracts: 'العقود',
    navLogin: 'تسجيل الدخول',
    navRegister: 'إنشاء حساب',
    navDashboard: 'لوحة التحكم',
    navSignOut: 'تسجيل الخروج',
    
    homeTitle: 'حلول جذرية لصيانة منزلك',
    homeSubtitle: 'نحن لا نقدم مجرد خدمات، بل حلولاً جذرية ومستدامة. نسعى للحل وليس للاستبدال.',
    homeCta: 'اطلب خدمة الآن',
    
    emergencyTitle: 'خدمة الطوارئ 24/7',
    emergencySubtitle: 'جاهزون للمساعدة في حالات الطوارئ لجميع أعمال الصيانة المنزلية.',
    emergencyCta: 'اتصل الآن',

    servicesTitle: 'خدماتنا',
    service1: { title: 'كهرباء', desc: 'إصلاح الأعطال الكهربائية، تمديدات، تركيب إضاءة، وصيانة شاملة.' },
    service2: { title: 'سباكة', desc: 'تصليح تسربات المياه، تركيب خلاطات، وصيانة سخانات المياه.' },
    service3: { title: 'دهانات', desc: 'دهانات داخلية وخارجية، ترميم الجدران، وتصاميم ديكورية مميزة.' },
    service4: { title: 'مكيفات', desc: 'تركيب، صيانة، وتنظيف جميع أنواع المكيفات لضمان أفضل أداء.' },

    contractsTitle: 'عقود الصيانة السنوية',
    contractsSubtitle: 'نقدم حلولاً متكاملة لصيانة منزلك على مدار العام. عقودنا السنوية تضمن لك راحة البال من خلال الصيانة الوقائية والدورية.',
    contractsCta: 'اطلب عقد الآن',

    aboutTitle: 'من نحن؟',
    aboutText1: 'نحن فريق من الفنيين المتخصصين في كافة أعمال الصيانة المنزلية. هدفنا ليس فقط إصلاح الأعطال، بل تقديم حلول دائمة ومستدامة تضمن سلامة وراحة منزلك.',
    aboutText2: 'نؤمن بأن الصيانة ليست مجرد مهمة، بل هي فن يتطلب الدقة والخبرة والالتزام بالجودة. لذلك، نعمل بشفافية تامة ونضمن لك أفضل النتائج.',
    
    geminiTitle: 'مساعد الصيانة الذكي ✨',
    geminiSubtitle: 'اكتب مشكلة الصيانة التي تواجهها واحصل على حل مقترح فوري.',
    geminiPlaceholder: 'مثال: صنبور المطبخ يسرب الماء',
    geminiCta: 'احصل على حل',
    geminiResultTitle: 'الحل المقترح:',

    loginTitle: 'تسجيل الدخول',
    loginEmailPlaceholder: 'البريد الإلكتروني',
    loginPasswordPlaceholder: 'كلمة المرور',
    loginBtn: 'تسجيل الدخول',
    loginNoAccount: 'لا تمتلك حسابًا؟',
    loginCreateAccount: 'إنشاء حساب جديد',

    contactTitle: 'تواصل معنا',
    contactSubtitle: 'يسعدنا تلقي استفساراتكم واقتراحاتكم. سيقوم فريقنا بالرد عليكم في أقرب وقت ممكن.',
    contactName: 'الاسم الكامل',
    contactEmail: 'البريد الإلكتروني',
    contactMessage: 'رسالتك',
    contactSendBtn: 'إرسال الرسالة',
    contactWhatsappBtn: 'تواصل واتساب',
    contactSuccess: 'تم إرسال رسالتك بنجاح!',
    contactError: 'عذراً، حدث خطأ أثناء إرسال الرسالة. يرجى المحاولة مرة أخرى.',

    contactInfoTitle: 'معلومات الاتصال',
    contactInfoEmail: 'البريد الإلكتروني:',
    contactInfoEmailValue: 'abuashraf@gmail.com',
    contactInfoPhone: 'رقم الهاتف:',
    contactInfoPhoneValue: '+971 50 703 0843',

    dashboardTitle: 'لوحة تحكم المستخدم',
    dashboardWelcome: 'مرحباً بك،',
    dashboardYourId: 'معرف المستخدم الخاص بك (لمشاركة البيانات):',
    dashboardTasks: 'مهامك:',
    dashboardMessages: 'رسائل العملاء:',
    dashboardAddTask: 'إضافة مهمة جديدة',
    dashboardTaskPlaceholder: 'اكتب مهمة جديدة...',
    dashboardAddBtn: 'إضافة',
    
    ceoName: 'أبو أشرف',
    ceoTitle: 'الرئيس التنفيذي',
    
    fallbackError: 'عذراً، حدث خطأ ما. يرجى المحاولة مرة أخرى.',
    networkError: 'حدث خطأ في الشبكة. يرجى التحقق من اتصالك.'
  },
  en: {
    lang: 'en',
    dir: 'ltr',
    companyName: 'Sadeed',
    title: 'Sadeed Maintenance',
    navHome: 'Home',
    navServices: 'Services',
    navAbout: 'About Us',
    navContracts: 'Contracts',
    navLogin: 'Login',
    navRegister: 'Register',
    navDashboard: 'Dashboard',
    navSignOut: 'Sign Out',

    homeTitle: 'Lasting Solutions for Your Home Maintenance',
    homeSubtitle: 'We offer more than just services; we provide sustainable and permanent solutions. We seek a fix, not a replacement.',
    homeCta: 'Request a Service Now',

    emergencyTitle: '24/7 Emergency Service',
    emergencySubtitle: 'We are ready to assist with all emergency home maintenance tasks.',
    emergencyCta: 'Call Now',

    servicesTitle: 'Our Services',
    service1: { title: 'Electricity', desc: 'Repairing electrical faults, installations, lighting, and comprehensive maintenance.' },
    service2: { title: 'Plumbing', desc: 'Fixing water leaks, installing mixers, and maintaining water heaters.' },
    service3: { title: 'Painting', desc: 'Interior and exterior painting, wall restoration, and unique decorative designs.' },
    service4: { title: 'Air Conditioning', desc: 'Installation, maintenance, and cleaning of all types of air conditioners for optimal performance.' },
    
    contractsTitle: 'Annual Maintenance Contracts',
    contractsSubtitle: 'We offer comprehensive solutions for your home\'s maintenance throughout the year. Our annual contracts guarantee peace of mind through preventive and periodic maintenance.',
    contractsCta: 'Request a Contract Now',

    aboutTitle: 'About Us',
    aboutText1: 'We are a team of specialized technicians in all home maintenance works. Our goal is not just to fix faults, but to provide permanent and sustainable solutions that ensure the safety and comfort of your home.',
    aboutText2: 'We believe that maintenance is not just a task, but an art that requires precision, experience, and a commitment to quality. Therefore, we work with complete transparency and guarantee the best results.',
    
    geminiTitle: 'Smart Maintenance Assistant ✨',
    geminiSubtitle: 'Describe the maintenance issue you are facing and get an instant suggested solution.',
    geminiPlaceholder: 'e.g. The kitchen faucet is leaking',
    geminiCta: 'Get a Solution',
    geminiResultTitle: 'Suggested Solution:',

    loginTitle: 'Login',
    loginEmailPlaceholder: 'Email Address',
    loginPasswordPlaceholder: 'Password',
    loginBtn: 'Login',
    loginNoAccount: 'Don\'t have an account?',
    loginCreateAccount: 'Create a new account',

    registerTitle: 'Create a New Account',
    registerEmailPlaceholder: 'Email Address',
    registerPasswordPlaceholder: 'Password (at least 6 characters)',
    registerBtn: 'Create Account',
    registerHasAccount: 'Already have an account?',
    registerLoginHere: 'Login here',

    contactTitle: 'Contact Us',
    contactSubtitle: 'We are happy to receive your inquiries and suggestions. Our team will get back to you as soon as possible.',
    contactName: 'Full Name',
    contactEmail: 'Email Address',
    contactMessage: 'Your Message',
    contactSendBtn: 'Send Message',
    contactWhatsappBtn: 'Contact via WhatsApp',
    contactSuccess: 'Your message has been sent successfully!',
    contactError: 'Sorry, an error occurred while sending the message. Please try again.',

    contactInfoTitle: 'Contact Information',
    contactInfoEmail: 'Email:',
    contactInfoEmailValue: 'abuashraf@gmail.com',
    contactInfoPhone: 'Phone:',
    contactInfoPhoneValue: '+971 50 703 0843',

    dashboardTitle: 'User Dashboard',
    dashboardWelcome: 'Welcome,',
    dashboardYourId: 'Your User ID (for data sharing):',
    dashboardTasks: 'Your Tasks:',
    dashboardMessages: 'Customer Messages:',
    dashboardAddTask: 'Add a new task',
    dashboardTaskPlaceholder: 'Write a new task...',
    dashboardAddBtn: 'Add',
    
    ceoName: 'Abu Ashraf',
    ceoTitle: 'CEO',

    fallbackError: 'Sorry, something went wrong. Please try again.',
    networkError: 'A network error occurred. Please check your connection.'
  },
};

const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [authReady, setAuthReady] = useState(false);
  const [lang, setLang] = useState('ar');

  useEffect(() => {
    const unsub = onAuthStateChanged(auth, async (user) => {
      setUser(user);
      setAuthReady(true);
    });

    const signIn = async () => {
      try {
        if (initialAuthToken) {
          await signInWithCustomToken(auth, initialAuthToken);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Firebase Auth Error:", error);
      }
    };
    signIn();

    return () => unsub();
  }, []);

  useEffect(() => {
    const storedLang = localStorage.getItem('lang');
    if (storedLang && storedLang in translations) {
      setLang(storedLang);
    } else {
      localStorage.setItem('lang', 'ar');
    }
  }, []);

  const toggleLang = () => {
    const newLang = lang === 'ar' ? 'en' : 'ar';
    setLang(newLang);
    localStorage.setItem('lang', newLang);
    document.documentElement.dir = translations[newLang].dir;
    document.documentElement.lang = newLang;
  };

  const contextValue = { user, authReady, lang, toggleLang, texts: translations[lang] };

  return (
    <AuthContext.Provider value={contextValue}>
      {children}
    </AuthContext.Provider>
  );
};

const Header = ({ navigate }) => {
  const { user, texts, toggleLang, lang } = React.useContext(AuthContext);

  return (
    <header className="bg-white shadow-lg rounded-b-2xl p-4 md:p-6 mb-12">
      <nav className="container mx-auto flex justify-between items-center flex-wrap">
        <button onClick={toggleLang} className="p-2 bg-[#2C3E50] text-white rounded-full shadow-lg">
          {lang === 'ar' ? 'EN' : 'AR'}
        </button>
        <a href="#" className="flex items-center">
          <p className="text-2xl font-bold text-[#2C3E50]">{texts.companyName}</p>
        </a>
        <ul className="flex space-x-6">
          <li><button onClick={() => navigate('home')} className="text-gray-600 hover:text-[#3498DB] font-semibold transition-all-300">{texts.navHome}</button></li>
          <li><button onClick={() => navigate('services')} className="text-gray-600 hover:text-[#3498DB] font-semibold transition-all-300">{texts.navServices}</button></li>
          <li><button onClick={() => navigate('about')} className="text-gray-600 hover:text-[#3498DB] font-semibold transition-all-300">{texts.navAbout}</button></li>
          <li><button onClick={() => navigate('contact')} className="text-gray-600 hover:text-[#3498DB] font-semibold transition-all-300">{texts.navContact}</button></li>
          {user ? (
            <>
              <li><button onClick={() => navigate('dashboard')} className="text-gray-600 hover:text-[#3498DB] font-semibold transition-all-300">{texts.navDashboard}</button></li>
              <li><button onClick={() => signOut(auth)} className="text-gray-600 hover:text-[#3498DB] font-semibold transition-all-300">{texts.navSignOut}</button></li>
            </>
          ) : (
            <>
              <li><button onClick={() => navigate('login')} className="text-gray-600 hover:text-[#3498DB] font-semibold transition-all-300">{texts.navLogin}</button></li>
              <li><button onClick={() => navigate('register')} className="text-gray-600 hover:text-[#3498DB] font-semibold transition-all-300">{texts.navRegister}</button></li>
            </>
          )}
        </ul>
      </nav>
    </header>
  );
};

const Footer = () => {
  const { texts } = React.useContext(AuthContext);
  const whatsappUrl = `https://wa.me/${texts.contactInfoPhoneValue.replace(/\s/g, '')}`;
  return (
    <footer className="bg-[#2C3E50] text-white rounded-t-2xl p-6 md:p-8 text-center mt-12 shadow-inner">
      <div className="container mx-auto">
        <p className="mb-4 text-lg" dangerouslySetInnerHTML={{ __html: texts.footerText }}></p>
        <div className="flex justify-center space-x-6 text-xl mb-4">
          <a href="#" className="hover:text-[#3498DB] transition-all-300"><i className="fab fa-facebook-f"></i></a>
          <a href="#" className="hover:text-[#3498DB] transition-all-300"><i className="fab fa-twitter"></i></a>
          <a href="#" className="hover:text-[#3498DB] transition-all-300"><i className="fab fa-instagram"></i></a>
          <a href={whatsappUrl} target="_blank" rel="noopener noreferrer" className="hover:text-[#3498DB] transition-all-300"><i className="fab fa-whatsapp"></i></a>
        </div>
        <div className="text-sm font-light">
          <p className="mb-1">{texts.contactInfoPhone} <a href={`tel:${texts.contactInfoPhoneValue}`} className="hover:underline">{texts.contactInfoPhoneValue}</a></p>
          <p>{texts.contactInfoEmail} <a href={`mailto:${texts.contactInfoEmailValue}`} className="hover:underline">{texts.contactInfoEmailValue}</a></p>
        </div>
      </div>
    </footer>
  );
};

const HomePage = ({ navigate }) => {
  const { texts } = React.useContext(AuthContext);
  const phoneUrl = `tel:${texts.contactInfoPhoneValue.replace(/\s/g, '')}`;
  return (
    <>
      <section className="bg-[#2C3E50] text-white rounded-3xl p-8 md:p-20 text-center mb-12 shadow-2xl relative overflow-hidden">
        <h1 className="text-4xl md:text-6xl font-bold mb-4 leading-tight">{texts.homeTitle}</h1>
        <p className="text-lg md:text-xl max-w-3xl mx-auto mb-8 font-light">{texts.homeSubtitle}</p>
        <button onClick={() => navigate('services')} className="bg-white text-[#3498DB] font-extrabold py-4 px-10 rounded-full shadow-lg hover:bg-gray-200 transition-all-300 transform hover:scale-105">{texts.homeCta}</button>
      </section>

      {/* قسم الطوارئ (جديد) */}
      <section className="bg-[#E74C3C] text-white rounded-3xl p-8 md:p-16 text-center mb-12 shadow-2xl">
        <h2 className="text-3xl md:text-5xl font-bold mb-4">{texts.emergencyTitle}</h2>
        <p className="text-lg md:text-xl max-w-3xl mx-auto mb-8 font-light">{texts.emergencySubtitle}</p>
        <a href={phoneUrl} className="bg-white text-[#E74C3C] font-extrabold py-4 px-10 rounded-full shadow-lg hover:bg-gray-200 transition-all-300 transform hover:scale-105">
          <i className="fas fa-phone-alt"></i> {texts.emergencyCta}
        </a>
      </section>

      <ServicesSection />
      <ContractsSection />
    </>
  );
};

const ServicesSection = () => {
  const { texts } = React.useContext(AuthContext);
  return (
    <section className="mb-12">
      <h2 className="text-center text-3xl font-bold text-[#2C3E50] mb-8">{texts.servicesTitle}</h2>
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-8">
        {[texts.service1, texts.service2, texts.service3, texts.service4].map((service, index) => (
          <div key={index} className="bg-white rounded-2xl shadow-xl p-8 text-center hover:shadow-2xl hover:scale-105 transition-all-300 cursor-pointer">
            <div className="text-[#3498DB] text-5xl mb-4 flex justify-center">
              {index === 0 && (
                <svg className="h-12 w-12 text-[#2C3E50]"  width="24" height="24" viewBox="0 0 24 24" strokeWidth="2" stroke="currentColor" fill="none" strokeLinecap="round" strokeLinejoin="round">  <path stroke="none" d="M0 0h24v24H0z"/>  <polyline points="13 3 13 10 19 10 11 21 11 14 5 14 13 3" /></svg>
              )}
              {index === 1 && (
                <svg className="h-12 w-12 text-[#2C3E50]"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  strokeWidth="2"  strokeLinecap="round"  strokeLinejoin="round">  <path d="M12 2L9.5 6.5L14 10L9.5 13.5L12 18L14.5 13.5L10 10L14.5 6.5L12 2Z" />  <path d="M12 2V22" />  <path d="M2 12H22" /></svg>
              )}
              {index === 2 && (
                <svg className="h-12 w-12 text-[#2C3E50]"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  strokeWidth="2"  strokeLinecap="round"  strokeLinejoin="round">  <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5A5.5 5.5 0 0 1 7.5 3c1.74 0 3.41.81 4.5 2.05A5.5 5.5 0 0 1 16.5 3c3.07 0 5.5 2.43 5.5 5.5 0 3.78-3.4 6.86-8.55 11.54L12 21.35z" /></svg>
              )}
              {index === 3 && (
                <svg className="h-12 w-12 text-[#2C3E50]"  viewBox="0 0 24 24"  fill="none"  stroke="currentColor"  strokeWidth="2"  strokeLinecap="round"  strokeLinejoin="round">  <line x1="12" y1="1" x2="12" y2="23" />  <path d="M17 5H7C4.79 5 3 6.79 3 9v6c0 2.21 1.79 4 4 4h10c2.21 0 4-1.79 4-4V9c0-2.21-1.79-4-4-4z" /></svg>
              )}
            </div>
            <h3 className="text-2xl font-bold mb-2">{service.title}</h3>
            <p className="text-gray-600">{service.desc}</p>
          </div>
        ))}
      </div>
    </section>
  );
};

const ContractsSection = () => {
    const { texts } = React.useContext(AuthContext);
    return (
        <section className="bg-[#2C3E50] text-white rounded-3xl p-8 md:p-16 text-center mb-12 shadow-2xl">
            <h2 className="text-3xl md:text-5xl font-bold mb-4">{texts.contractsTitle}</h2>
            <p className="text-lg md:text-xl max-w-3xl mx-auto mb-8 font-light">{texts.contractsSubtitle}</p>
            <a href="#" className="bg-[#3498DB] text-white font-extrabold py-4 px-10 rounded-full shadow-lg hover:bg-gray-800 transition-all-300 transform hover:scale-105">{texts.contractsCta}</a>
        </section>
    )
}

const AboutSection = () => {
    const { texts } = React.useContext(AuthContext);
    return (
      <section className="bg-white rounded-3xl p-8 md:p-16 mb-12 shadow-xl">
        <h2 className="text-center text-3xl font-bold text-[#2C3E50] mb-8">{texts.aboutTitle}</h2>
        <div className="max-w-4xl mx-auto text-center">
          <p className="text-lg text-gray-600 leading-relaxed mb-4">{texts.aboutText1}</p>
          <p className="text-lg text-gray-600 leading-relaxed">{texts.aboutText2}</p>
        </div>
      </section>
    );
};

const ContactPage = () => {
    const { texts } = React.useContext(AuthContext);
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');
    const [message, setMessage] = useState('');
    const [status, setStatus] = useState('');
    const whatsappUrl = `https://wa.me/${texts.contactInfoPhoneValue.replace(/\s/g, '')}`;

    const handleSubmit = async (e) => {
        e.preventDefault();
        setStatus('sending');
        try {
            const messagesCollection = collection(db, 'artifacts', appId, 'public', 'data', 'messages');
            await addDoc(messagesCollection, {
                name,
                email,
                message,
                timestamp: new Date()
            });
            setStatus('success');
            setName('');
            setEmail('');
            setMessage('');
            setTimeout(() => setStatus(''), 5000);
        } catch (error) {
            console.error("Error submitting message:", error);
            setStatus('error');
            setTimeout(() => setStatus(''), 5000);
        }
    };

    return (
        <section className="bg-white rounded-3xl p-8 md:p-16 mb-12 shadow-xl">
            <h2 className="text-center text-3xl font-bold text-[#2C3E50] mb-4">{texts.contactTitle}</h2>
            <p className="text-center text-lg text-gray-600 mb-8">{texts.contactSubtitle}</p>
            <div className="max-w-xl mx-auto">
                <div className="mb-8 p-6 bg-gray-100 rounded-xl">
                    <h3 className="text-lg font-bold text-[#2C3E50] mb-4">{texts.contactInfoTitle}</h3>
                    <p className="text-gray-700 font-semibold mb-2">{texts.contactInfoPhone} <a href={`tel:${texts.contactInfoPhoneValue}`} className="font-normal text-[#2C3E50] hover:underline">{texts.contactInfoPhoneValue}</a></p>
                    <p className="text-gray-700 font-semibold mb-4">{texts.contactInfoEmail} <a href={`mailto:${texts.contactInfoEmailValue}`} className="font-normal text-[#2C3E50] hover:underline">{texts.contactInfoEmailValue}</a></p>
                    <a href={whatsappUrl} target="_blank" rel="noopener noreferrer" className="inline-block bg-[#25D366] text-white font-bold py-3 px-6 rounded-full shadow-lg hover:bg-[#128C7E] transition-all-300">
                        <i className="fab fa-whatsapp"></i> {texts.contactWhatsappBtn}
                    </a>
                </div>
                <form onSubmit={handleSubmit}>
                    <div className="mb-4">
                        <label htmlFor="name" className="block text-gray-700 font-semibold mb-2">{texts.contactName}</label>
                        <input
                            type="text"
                            id="name"
                            value={name}
                            onChange={(e) => setName(e.target.value)}
                            className="w-full p-3 rounded-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-[#3498DB] transition-all-300"
                            required
                        />
                    </div>
                    <div className="mb-4">
                        <label htmlFor="email" className="block text-gray-700 font-semibold mb-2">{texts.contactEmail}</label>
                        <input
                            type="email"
                            id="email"
                            value={email}
                            onChange={(e) => setEmail(e.target.value)}
                            className="w-full p-3 rounded-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-[#3498DB] transition-all-300"
                            required
                        />
                    </div>
                    <div className="mb-6">
                        <label htmlFor="message" className="block text-gray-700 font-semibold mb-2">{texts.contactMessage}</label>
                        <textarea
                            id="message"
                            value={message}
                            onChange={(e) => setMessage(e.target.value)}
                            rows="5"
                            className="w-full p-3 rounded-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-[#3498DB] transition-all-300"
                            required
                        ></textarea>
                    </div>
                    <button
                        type="submit"
                        disabled={status === 'sending'}
                        className="w-full bg-[#3498DB] text-white font-bold py-3 px-6 rounded-full shadow-lg hover:bg-[#2C3E50] transition-all-300 transform hover:scale-105 disabled:bg-gray-400 disabled:cursor-not-allowed"
                    >
                        {status === 'sending' ? 'جارٍ الإرسال...' : texts.contactSendBtn}
                    </button>
                    {status === 'success' && <div className="mt-4 p-4 text-center bg-green-100 text-green-700 rounded-xl">{texts.contactSuccess}</div>}
                    {status === 'error' && <div className="mt-4 p-4 text-center bg-red-100 text-red-700 rounded-xl">{texts.contactError}</div>}
                </form>
            </div>
        </section>
    );
};

const LoginRegisterPage = ({ navigate, isRegister }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const { texts } = React.useContext(AuthContext);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    try {
      if (isRegister) {
        await createUserWithEmailAndPassword(auth, email, password);
      } else {
        await signInWithEmailAndPassword(auth, email, password);
      }
    } catch (err) {
      setError(err.message);
    }
  };

  return (
    <div className="flex justify-center items-center h-screen bg-gray-100">
      <div className="bg-white p-8 rounded-2xl shadow-xl w-full max-w-md">
        <h2 className="text-center text-3xl font-bold text-[#2C3E50] mb-8">{isRegister ? texts.registerTitle : texts.loginTitle}</h2>
        {error && <div className="bg-red-100 text-red-700 p-4 rounded-xl mb-4">{error}</div>}
        <form onSubmit={handleSubmit}>
          <div className="mb-4">
            <input
              type="email"
              placeholder={isRegister ? texts.registerEmailPlaceholder : texts.loginEmailPlaceholder}
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              className="w-full p-3 rounded-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-[#3498DB]"
              required
            />
          </div>
          <div className="mb-6">
            <input
              type="password"
              placeholder={isRegister ? texts.registerPasswordPlaceholder : texts.loginPasswordPlaceholder}
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="w-full p-3 rounded-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-[#3498DB]"
              required
            />
          </div>
          <button type="submit" className="w-full bg-[#3498DB] text-white font-bold py-3 px-6 rounded-full shadow-lg hover:bg-[#2C3E50] transition-all-300 transform hover:scale-105">
            {isRegister ? texts.registerBtn : texts.loginBtn}
          </button>
        </form>
        <div className="mt-4 text-center">
          {isRegister ? (
            <p>{texts.registerHasAccount} <button onClick={() => navigate('login')} className="text-[#3498DB] hover:underline">{texts.registerLoginHere}</button></p>
          ) : (
            <p>{texts.loginNoAccount} <button onClick={() => navigate('register')} className="text-[#3498DB] hover:underline">{texts.loginCreateAccount}</button></p>
          )}
        </div>
      </div>
    </div>
  );
};

const DashboardPage = () => {
    const { user, texts } = React.useContext(AuthContext);
    const [tasks, setTasks] = useState([]);
    const [messages, setMessages] = useState([]);
    const [error, setError] = useState('');

    useEffect(() => {
        if (!user) return;
        // Fetch tasks for the current user in real-time
        const userId = user.uid;
        const tasksQuery = query(collection(db, 'artifacts', appId, 'users', userId, 'tasks'));
        const unsubTasks = onSnapshot(tasksQuery, (querySnapshot) => {
            const fetchedTasks = [];
            // Sort tasks by createdAt date descending
            querySnapshot.forEach((doc) => {
                fetchedTasks.push({ ...doc.data(), id: doc.id });
            });
            setTasks(fetchedTasks.sort((a,b) => b.createdAt - a.createdAt));
        }, (err) => {
            setError(err.message);
            console.error("Firestore Error:", err);
        });

        // Fetch public messages in real-time
        const messagesQuery = query(collection(db, 'artifacts', appId, 'public', 'data', 'messages'), orderBy('timestamp', 'desc'));
        const unsubMessages = onSnapshot(messagesQuery, (querySnapshot) => {
            const fetchedMessages = [];
            querySnapshot.forEach((doc) => {
                fetchedMessages.push({ ...doc.data(), id: doc.id });
            });
            setMessages(fetchedMessages);
        }, (err) => {
            setError(err.message);
            console.error("Firestore Error:", err);
        });

        return () => {
            unsubTasks();
            unsubMessages();
        };
    }, [user]);

    const handleAddTask = async () => {
        if (!newTask.trim()) return;
        setError('');
        try {
            const userId = user.uid;
            await addDoc(collection(db, 'artifacts', appId, 'users', userId, 'tasks'), {
                text: newTask,
                completed: false,
                createdAt: new Date()
            });
            setNewTask('');
        } catch (err) {
            setError(err.message);
        }
    };
    
    const [newTask, setNewTask] = useState('');

    if (!user) {
        return <div className="text-center mt-12 text-gray-500">الرجاء تسجيل الدخول لعرض لوحة التحكم.</div>;
    }

    return (
        <div className="p-8 bg-white rounded-3xl shadow-xl">
            <h2 className="text-3xl font-bold text-[#2C3E50] mb-4">{texts.dashboardTitle}</h2>
            <p className="text-xl text-gray-700 mb-2">{texts.dashboardWelcome} {user.email}!</p>
            <p className="text-sm text-gray-500 mb-6 break-words">{texts.dashboardYourId} {user.uid}</p>
            
            {error && <div className="bg-red-100 text-red-700 p-4 rounded-xl mb-4">{error}</div>}

            <h3 className="text-2xl font-bold text-[#2C3E50] mt-8 mb-4">{texts.dashboardTasks}</h3>
            <div className="flex mb-4">
                <input
                    type="text"
                    value={newTask}
                    onChange={(e) => setNewTask(e.target.value)}
                    placeholder={texts.dashboardTaskPlaceholder}
                    className="flex-grow p-3 rounded-l-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-[#3498DB]"
                />
                <button onClick={handleAddTask} className="bg-[#3498DB] text-white font-bold py-3 px-6 rounded-r-xl hover:bg-[#2C3E50] transition-all-300">{texts.dashboardAddBtn}</button>
            </div>
            
            <ul className="bg-gray-100 rounded-xl p-4">
                {tasks.length > 0 ? (
                    tasks.map(task => (
                        <li key={task.id} className="p-2 border-b border-gray-200 last:border-b-0">
                            {task.text}
                        </li>
                    ))
                ) : (
                    <li className="text-gray-500">لا توجد مهام جديدة.</li>
                )}
            </ul>

            <h3 className="text-2xl font-bold text-[#2C3E50] mt-8 mb-4">{texts.dashboardMessages}</h3>
            <ul className="bg-gray-100 rounded-xl p-4">
                {messages.length > 0 ? (
                    messages.map(message => (
                        <li key={message.id} className="p-4 border-b border-gray-200 last:border-b-0">
                            <p className="font-bold text-gray-800">{message.name} ({message.email})</p>
                            <p className="text-gray-600 mt-1">{message.message}</p>
                            <p className="text-sm text-gray-400 mt-2">{new Date(message.timestamp.toDate()).toLocaleString()}</p>
                        </li>
                    ))
                ) : (
                    <li className="text-gray-500">لا توجد رسائل جديدة.</li>
                )}
            </ul>
        </div>
    );
};

const MainContent = () => {
  const [page, setPage] = useState('home');
  const { authReady } = React.useContext(AuthContext);

  const navigate = (newPage) => setPage(newPage);
  
  if (!authReady) {
    return (
      <div className="flex justify-center items-center h-screen">
        <i className="fas fa-spinner fa-spin text-4xl text-[#2C3E50]"></i>
      </div>
    );
  }

  return (
    <div className="min-h-screen flex flex-col">
      <Header navigate={navigate} />
      <main className="container mx-auto px-4 flex-grow">
        {(() => {
          switch (page) {
            case 'home':
              return <HomePage navigate={navigate} />;
            case 'services':
              return <ServicesSection />;
            case 'about':
              return <AboutSection />;
            case 'contact':
              return <ContactPage />;
            case 'login':
              return <LoginRegisterPage navigate={navigate} isRegister={false} />;
            case 'register':
              return <LoginRegisterPage navigate={navigate} isRegister={true} />;
            case 'dashboard':
              return <DashboardPage />;
            default:
              return <HomePage navigate={navigate} />;
          }
        })()}
      </main>
      <Footer />
    </div>
  );
};

const App = () => {
  return (
    <AuthProvider>
      <MainContent />
    </AuthProvider>
  );
};

export default App;
