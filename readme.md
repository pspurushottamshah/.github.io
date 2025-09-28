import React, { useState, useEffect } from 'react';
import { Briefcase, ThumbsUp, Linkedin, Github, Mail, Code, Cloud, Server, Users, MessageSquare, Twitter } from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, doc, onSnapshot, collection, setDoc, getDocs } from 'firebase/firestore';
import { setLogLevel } from 'firebase/firestore';

// Set Firebase log level to debug for better visibility in the console
setLogLevel('debug');

// --- Firebase Initialization and Globals ---
// Firebase configuration using global variables from the environment
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

// Initialize Firebase App, Firestore, and Auth instances
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const auth = getAuth(app);

// Data to be seeded to Firestore on first run
const myInfo = {
  profile: {
    name: 'Jane Doe',
    title: 'Senior Project Manager',
    bio: 'A results-driven Senior Project Manager with a passion for leading cross-functional teams and delivering complex projects on time and within budget. I specialize in a range of technologies from enterprise software like Dynamics 365 and Salesforce to custom web solutions and data center infrastructure.',
    socials: {
      linkedin: '#',
      twitter: '#',
      github: '#'
    },
    profileUrl: 'https://placehold.co/500x500/18181B/FBBF24?text=JD'
  },
  // Added IDs for cleaner Firebase interaction (though not strictly necessary for seeding)
  skills: [
    { name: 'Dynamics 365 / CRM', level: 95 },
    { name: 'Salesforce CRM', level: 90 },
    { name: 'ERP & F&O', level: 85 },
    { name: 'Data Center Infrastructure', level: 92 },
    { name: 'Custom Web & Portals', level: 98 },
  ],
  experience: [
    {
      title: 'Senior Project Manager',
      company: 'Enterprise Tech Solutions',
      duration: '2021 - Present',
      description: 'Led a team of 15 to successfully implement a Dynamics 365 CRM and Sales module for a major retail client, improving sales pipeline visibility by 40%.',
    },
    {
      title: 'Implementation Consultant',
      company: 'CRM Masters Inc.',
      duration: '2018 - 2021',
      description: 'Managed the full lifecycle of multiple Salesforce implementation projects, from discovery and design to deployment and user training.',
    },
    {
      title: 'Project Coordinator',
      company: 'Cloud Innovations',
      duration: '2015 - 2018',
      description: 'Assisted in the migration of on-premise data centers to a hybrid cloud environment, ensuring minimal downtime and data integrity.',
    },
  ],
  caseStudies: [
    {
      title: 'Dynamics 365 F&O Implementation',
      description: 'Oversaw the end-to-end implementation of Dynamics 365 Finance and Operations for a global manufacturing client. The project involved integrating new financial reporting tools and streamlining supply chain operations, leading to a 20% reduction in operational costs. I managed a cross-functional team of 25 consultants and developers, ensuring adherence to a strict timeline and budget.',
      tags: ['ERP', 'Dynamics 365', 'Finance', 'Supply Chain', 'Project Management'],
      logo: 'D365',
    },
    {
      title: 'Salesforce CRM Rollout',
      description: 'Managed the company-wide rollout of Salesforce Sales and Service Cloud to a team of over 500 sales and support agents. This project required extensive change management and custom development of a user portal. The successful launch resulted in a 30% increase in lead conversion rates and a significant improvement in customer satisfaction scores.',
      tags: ['Salesforce', 'CRM', 'Cloud', 'User Training', 'Customization'],
      logo: 'Salesforce',
    },
    {
      title: 'Custom Web Portal Development',
      description: 'Led the development of a bespoke web portal for a logistics company to track real-time freight and inventory. The project involved complex API integrations and a focus on intuitive UX/UI design. I guided a team of front-end and back-end developers from concept to launch, ensuring all technical and business requirements were met.',
      tags: ['Web Development', 'Custom Portal', 'API Integration', 'UX/UI', 'Agile'],
      logo: 'Web',
    },
    {
      title: 'Data Center Migration',
      description: 'Coordinated the physical and virtual migration of a company\'s on-premise data center to a new co-location facility. This high-stakes project involved meticulous planning and risk management to ensure zero data loss and minimal service interruption. I worked closely with network engineers and system administrators to deliver the project ahead of schedule.',
      tags: ['Infrastructure', 'Data Center', 'Cloud Migration', 'Risk Management', 'IT Operations'],
      logo: 'Server',
    },
  ],
  recommendations: [
    {
      text: 'Jane\'s expertise in managing complex Salesforce implementations is unparalleled. She consistently delivered results and her leadership was a huge asset to our team.',
      recommender: 'Sarah Thompson',
      title: 'Head of Sales, TechCorp',
    },
    {
      text: 'Her deep understanding of Dynamics 365 coupled with her exceptional project management skills made our F&O implementation project a massive success. Highly recommended.',
      recommender: 'Michael Lee',
      title: 'CFO, Global Logistics Inc.',
    },
    {
      text: 'Jane is a natural leader who excels at bridging the gap between technical teams and business stakeholders. Her work on our custom portal was a game changer.',
      recommender: 'David Chen',
      title: 'Lead Developer, Innovate Solutions',
    },
  ],
};

// --- Utility Components ---

// SVG icons for technology
const TechLogo = ({ name, className }) => {
  switch (name) {
    case 'D365':
      return (
        // Custom SVG for Dynamics 365
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 48 48" className={className}>
          <path fill="#26A0DA" d="M19.3 12.8c.2-.2.3-.6.1-.8L14.4 7c-.2-.2-.6-.2-.8 0l-4.9 5.1c-.2.2-.2.6 0 .8l4.9 5.1c.2.2.6.2.8 0L19.3 12.8zm-1.8 1.8l-3.9 4-3.9-4 3.9-4 3.9 4z"/>
          <path fill="#1982BA" d="M28.7 12.8c-.2-.2-.6-.2-.8 0l-4.9 5.1c-.2.2-.3.6-.1.8l4.9 5.1c.2.2.6.2.8 0l4.9-5.1c.2-.2.2-.6 0-.8L28.7 12.8zm1.8 1.8l3.9 4 3.9-4-3.9-4-3.9 4z"/>
          <path fill="#1667A5" d="M19.3 26.2c.2.2.2.6 0 .8l-4.9 5.1c-.2.2-.6.2-.8 0l-4.9-5.1c-.2-.2-.2-.6 0-.8l4.9-5.1c.2-.2.6-.2.8 0L19.3 26.2zm-1.8-1.8l-3.9-4-3.9 4 3.9 4 3.9-4z"/>
          <path fill="#0F4687" d="M28.7 26.2c-.2.2-.6.2-.8 0l-4.9-5.1c-.2-.2-.2-.6 0-.8l4.9-5.1c.2-.2.6-.2.8 0l4.9 5.1c.2.2.2.6 0 .8l-4.9 5.1zm1.8-1.8l-3.9-4-3.9 4 3.9 4 3.9-4z"/>
          <circle fill="#fff" cx="24" cy="24" r="2.5"/>
        </svg>
      );
    case 'Salesforce':
      return (
        // Custom SVG for Salesforce
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" className={className}>
          <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm3.55 13.91l-1.42 1.42a1 1 0 01-1.41 0L11.7 15.6l-1.42 1.41a1 1 0 01-1.41 0L7.45 15.9l-1.42 1.42a1 1 0 01-1.41-1.41l1.42-1.42L4.62 12a1 1 0 010-1.41l1.42-1.42L4.62 8.41a1 1 0 011.41-1.41l1.42 1.42L8.41 7.45a1 1 0 011.41 0l1.42 1.42 1.41-1.42a1 1 0 011.41 0l1.42 1.42 1.42-1.42a1 1 0 011.41 1.41L15.9 9.88l1.42 1.42a1 1 0 010 1.41l-1.42 1.42 1.42 1.42a1 1 0 01-1.41 1.41z" fill="#00A1E0"/>
          <circle cx="12" cy="12" r="3" fill="#fff"/>
        </svg>
      );
    case 'Web':
      return <Code className={className} />;
    case 'Server':
      return <Server className={className} />;
    default:
      return <Briefcase className={className} />;
  }
};

const Modal = ({ isOpen, content, onClose }) => {
  if (!isOpen || !content) return null;

  const renderContent = () => {
    if (content.type === 'caseStudy') {
      return (
        <>
          <h3 className="text-3xl font-bold text-amber-500 mb-2">{content.title}</h3>
          <p className="text-gray-300 mb-4">{content.description}</p>
          <div className="flex flex-wrap gap-2 mb-4">
            {content.tags.map((tag, i) => (
              <span key={i} className="bg-amber-900 text-amber-100 text-xs font-semibold px-2 py-1 rounded-full">
                {tag}
              </span>
            ))}
          </div>
        </>
      );
    } else if (content.type === 'recommendation') {
      return (
        <>
          <p className="italic text-gray-300 text-lg mb-4">"{content.text}"</p>
          <div className="flex items-center space-x-4">
            <div className="w-12 h-12 rounded-full bg-amber-600 text-white flex items-center justify-center font-bold text-2xl">
              {content.recommender.charAt(0)}
            </div>
            <div>
              <h4 className="font-bold text-white">{content.recommender}</h4>
              <p className="text-sm text-gray-400">{content.title}</p>
            </div>
          </div>
        </>
      );
    }
  };

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-gray-900 bg-opacity-75 p-4" onClick={onClose}>
      <div 
        className="bg-neutral-800 rounded-3xl shadow-2xl max-w-2xl w-full p-8 relative transform transition-all duration-300 scale-100" 
        onClick={(e) => e.stopPropagation()} // Prevent click-through closing
      >
        <button onClick={onClose} className="absolute top-4 right-4 p-2 rounded-full bg-gray-700 text-gray-400 hover:bg-gray-600 transition-colors">
          <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
        <div className="mt-4">
          {renderContent()}
        </div>
      </div>
    </div>
  );
};

// --- Firebase Data Seeding ---

const seedDatabase = async (userId) => {
  // Check if data is already seeded for this user/app combo
  const isSeeded = localStorage.getItem(`seeded_${userId}_${appId}`);
  if (isSeeded === 'true') {
    // console.log("Data already seeded for user:", userId);
    return;
  }
  
  try {
    // 1. Profile
    const profileRef = doc(db, `artifacts/${appId}/users/${userId}/profile/info`);
    await setDoc(profileRef, myInfo.profile);

    // 2. Collections (Experience, Case Studies, Recommendations)
    const collections = {
      experience: myInfo.experience,
      caseStudies: myInfo.caseStudies,
      recommendations: myInfo.recommendations
    };

    for (const [key, items] of Object.entries(collections)) {
      const collectionRef = collection(db, `artifacts/${appId}/users/${userId}/${key}`);
      for (const item of items) {
        // Use addDoc to automatically generate unique IDs for collection items
        await setDoc(doc(collectionRef), item); 
      }
    }

    localStorage.setItem(`seeded_${userId}_${appId}`, 'true');
    console.log("Database successfully seeded.");
  } catch (error) {
    console.error("Error seeding database:", error);
  }
};


// --- Main Application Component ---

export const App = () => {
  const [currentPage, setCurrentPage] = useState('home');
  const [userData, setUserData] = useState({ profile: null, skills: myInfo.skills, experience: [], caseStudies: [], recommendations: [] });
  const [loading, setLoading] = useState(true);
  const [modalContent, setModalContent] = useState(null);
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [userId, setUserId] = useState(null);
  const [isAuthReady, setIsAuthReady] = useState(false);
  const [isSkillsVisible, setIsSkillsVisible] = useState(false);

  // 1. Authentication and Seeding
  useEffect(() => {
    const unsubscribeAuth = onAuthStateChanged(auth, async (user) => {
      let currentUserId = user?.uid;
      
      if (!user) {
        // Attempt custom token sign-in or anonymous sign-in
        const token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        if (token) {
          try {
            const credential = await signInWithCustomToken(auth, token);
            currentUserId = credential.user.uid;
          } catch (e) {
            console.error('Failed to sign in with custom token, signing in anonymously instead', e);
            const anonCredential = await signInAnonymously(auth);
            currentUserId = anonCredential.user.uid;
          }
        } else {
          const anonCredential = await signInAnonymously(auth);
          currentUserId = anonCredential.user.uid;
        }
      }
      
      if (currentUserId) {
        setUserId(currentUserId);
        // Seed the data only after successful auth
        await seedDatabase(currentUserId); 
      }
      setIsAuthReady(true);
    });

    return () => unsubscribeAuth();
  }, []);

  // 2. Data Fetching (using onSnapshot for real-time updates)
  useEffect(() => {
    if (!userId || !isAuthReady) return;

    // References
    const userProfileRef = doc(db, `artifacts/${appId}/users/${userId}/profile/info`);
    const caseStudiesCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/caseStudies`);
    const experienceCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/experience`);
    const recommendationsCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/recommendations`);

    // Profile listener
    const unsubscribeProfile = onSnapshot(userProfileRef, (docSnap) => {
      if (docSnap.exists()) {
        setUserData(prev => ({ ...prev, profile: docSnap.data() }));
      } else {
         // If profile doesn't exist (e.g., first run before seeding completes), use default info
         setUserData(prev => ({ ...prev, profile: myInfo.profile }));
      }
      setLoading(false);
    }, (error) => {
      console.error("Error fetching profile:", error);
      setLoading(false);
    });

    // Case Studies listener
    const unsubscribeCaseStudies = onSnapshot(caseStudiesCollectionRef, (snapshot) => {
      const caseStudies = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setUserData((prev) => ({ ...prev, caseStudies: caseStudies }));
    });

    // Experience listener
    const unsubscribeExperience = onSnapshot(experienceCollectionRef, (snapshot) => {
      const experience = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      // Sort experience by duration or some other metric if needed, but for now use Firestore order
      setUserData((prev) => ({ ...prev, experience: experience }));
    });

    // Recommendations listener
    const unsubscribeRecommendations = onSnapshot(recommendationsCollectionRef, (snapshot) => {
      const recommendations = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setUserData((prev) => ({ ...prev, recommendations: recommendations }));
    });

    // Cleanup listeners
    return () => {
      unsubscribeProfile();
      unsubscribeCaseStudies();
      unsubscribeExperience();
      unsubscribeRecommendations();
    };
  }, [userId, isAuthReady]);


  // 3. Skills Visibility Observer (for animation)
  useEffect(() => {
    // Only set up observer once authentication is ready and data might be loading
    if (!isAuthReady) return; 

    let observer;
    const skillsSection = document.getElementById('skills-section');
    
    if (skillsSection) {
        observer = new IntersectionObserver(
            (entries) => {
                entries.forEach((entry) => {
                    if (entry.isIntersecting) {
                        setIsSkillsVisible(true);
                        observer.unobserve(skillsSection); // Stop observing once visible
                    }
                });
            },
            { threshold: 0.5 } // Trigger when 50% of the section is visible
        );
        observer.observe(skillsSection);
    }
    
    return () => {
        if (observer) observer.disconnect();
    };
  }, [isAuthReady]);


  const openModal = (content) => {
    setModalContent(content);
    setIsModalOpen(true);
  };

  const closeModal = () => {
    setIsModalOpen(false);
    setModalContent(null);
  };

  if (loading) {
    return (
      <div className="flex items-center justify-center min-h-screen bg-neutral-900">
        <div className="flex flex-col items-center">
          <svg className="animate-spin h-8 w-8 text-amber-500 mb-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
            <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          <div className="text-xl font-semibold text-gray-300">
            Loading portfolio data...
          </div>
        </div>
      </div>
    );
  }

  const NavBar = () => (
    <nav className="fixed top-0 left-0 right-0 z-50 bg-neutral-950 shadow-xl p-4 rounded-b-xl border-b-4 border-amber-500">
      <div className="container mx-auto flex justify-center space-x-2 sm:space-x-8 overflow-x-auto whitespace-nowrap">
        <button
          onClick={() => setCurrentPage('home')}
          className={`flex items-center px-4 py-2 rounded-full transition-all duration-300 font-semibold text-sm sm:text-base ${
            currentPage === 'home'
              ? 'bg-amber-500 text-neutral-950 shadow-lg transform scale-105'
              : 'text-gray-300 hover:bg-gray-800 hover:text-amber-500'
          }`}
        >
          <Briefcase className="w-5 h-5 mr-1 sm:mr-2" />
          Home
        </button>
        <button
          onClick={() => setCurrentPage('caseStudies')}
          className={`flex items-center px-4 py-2 rounded-full transition-all duration-300 font-semibold text-sm sm:text-base ${
            currentPage === 'caseStudies'
              ? 'bg-amber-500 text-neutral-950 shadow-lg transform scale-105'
              : 'text-gray-300 hover:bg-gray-800 hover:text-amber-500'
          }`}
        >
          <Code className="w-5 h-5 mr-1 sm:mr-2" />
          Case Studies
        </button>
        <button
          onClick={() => setCurrentPage('recommendations')}
          className={`flex items-center px-4 py-2 rounded-full transition-all duration-300 font-semibold text-sm sm:text-base ${
            currentPage === 'recommendations'
              ? 'bg-amber-500 text-neutral-950 shadow-lg transform scale-105'
              : 'text-gray-300 hover:bg-gray-800 hover:text-amber-500'
          }`}
        >
          <MessageSquare className="w-5 h-5 mr-1 sm:mr-2" />
          Endorsements
        </button>
        <button
          onClick={() => setCurrentPage('contact')}
          className={`flex items-center px-4 py-2 rounded-full transition-all duration-300 font-semibold text-sm sm:text-base ${
            currentPage === 'contact'
              ? 'bg-amber-500 text-neutral-950 shadow-lg transform scale-105'
              : 'text-gray-300 hover:bg-gray-800 hover:text-amber-500'
          }`}
        >
          <Mail className="w-5 h-5 mr-1 sm:mr-2" />
          Contact
        </button>
      </div>
    </nav>
  );

  const HomePage = () => (
    <div className="container mx-auto px-4 py-16 pt-32">
      {/* Profile Section */}
      <div className="bg-neutral-800 rounded-3xl shadow-xl p-8 md:p-12 mb-12 border-t-4 border-amber-600">
        <div className="flex flex-col md:flex-row items-center md:items-start space-y-6 md:space-y-0 md:space-x-12">
          {/* Profile Image */}
          <div className="w-48 h-48 flex-shrink-0 rounded-full overflow-hidden shadow-2xl border-4 border-amber-500">
            <img src={userData.profile?.profileUrl || "https://placehold.co/500x500/18181B/FBBF24?text=JD"} alt="Profile" className="object-cover w-full h-full" />
          </div>
          {/* Profile Details */}
          <div className="text-center md:text-left flex-grow">
            <h1 className="text-4xl md:text-5xl font-extrabold text-white mb-2">
              {userData.profile?.name}
            </h1>
            <h2 className="text-xl text-amber-500 font-semibold mb-4">
              {userData.profile?.title}
            </h2>
            <p className="text-gray-300 max-w-2xl mx-auto md:mx-0 leading-relaxed">
              {userData.profile?.bio}
            </p>
            {/* Social Links */}
            <div className="mt-6 flex justify-center md:justify-start space-x-4">
              <a href={userData.profile?.socials?.linkedin} aria-label="LinkedIn" className="p-3 bg-gray-700 hover:bg-amber-500 rounded-full text-gray-400 hover:text-neutral-900 transition-colors">
                <Linkedin className="w-6 h-6" />
              </a>
              <a href={userData.profile?.socials?.twitter} aria-label="Twitter" className="p-3 bg-gray-700 hover:bg-amber-500 rounded-full text-gray-400 hover:text-neutral-900 transition-colors">
                <Twitter className="w-6 h-6" />
              </a>
              <a href={userData.profile?.socials?.github} aria-label="Github" className="p-3 bg-gray-700 hover:bg-amber-500 rounded-full text-gray-400 hover:text-neutral-900 transition-colors">
                <Github className="w-6 h-6" />
              </a>
            </div>
            <div className="mt-4 text-xs text-gray-600">
              User ID: {userId}
            </div>
          </div>
        </div>
      </div>

      {/* Core Competencies Section */}
      <div id="skills-section" className="bg-neutral-800 rounded-3xl shadow-xl p-8 md:p-12 mb-12">
        <h3 className="text-3xl font-bold text-white mb-6 text-center border-b pb-4 border-gray-700">Core Competencies</h3>
        <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
          {userData.skills?.map((skill, index) => (
            <div key={index} className="space-y-2">
              <div className="flex justify-between items-center text-gray-300 font-medium">
                <span>{skill.name}</span>
                <span>{skill.level}%</span>
              </div>
              <div className="w-full bg-gray-700 rounded-full h-3 overflow-hidden">
                <div
                  className={`bg-amber-500 h-3 rounded-full transition-all duration-1000 ease-out`}
{% comment %} 
    The loop below iterates over your skill data (e.g., in _data/skills.yml).
    Replace 'site.data.skills' with your actual data source if different.
{% endcomment %}

{% for skill in site.data.skills %}
<div class="my-4">
    <div class="flex justify-between items-center mb-1">
        <span class="text-sm font-semibold text-gray-700">{{ skill.name }}</span>
        <span class="text-xs text-gray-500">{{ skill.level }}%</span>
    </div>
    <div class="w-full bg-gray-200 rounded-full h-2.5">
        <!-- 
            FIXED LOGIC: Using {% if/else %} and closed correctly with {% endif %}.
            The error 'use endif' suggests a misplaced {% endfor %} elsewhere in your file.
        -->
        <div class="h-2.5 rounded-full bg-blue-600 transition-all duration-700 ease-in-out" 
             style="width: {% if page.isSkillsVisible %}{{ skill.level }}%{% else %}0%{% endif %};">
        </div>
    </div>
</div>
{% endfor %} 
<!-- This {% endfor %} closes the {% for skill in site.data.skills %} loop. -->

                  aria-valuenow={skill.level}
                  aria-valuemin="0"
                  aria-valuemax="100"
                  role="progressbar"
                ></div>
              </div>
            </div>
          ))}
        </div>
      </div>

      {/* Experience Timeline Section */}
      <div className="bg-neutral-800 rounded-3xl shadow-xl p-8 md:p-12">
        <h3 className="text-3xl font-bold text-white mb-6 text-center border-b pb-4 border-gray-700">Experience Timeline</h3>
        <div className="relative">
          <div className="absolute left-1/2 -ml-0.5 w-1 h-full bg-amber-500 rounded-full hidden md:block"></div>
          {userData.experience?.map((exp, index) => (
            <div key={index} className={`flex relative items-start mb-12 ${index % 2 === 0 ? 'md:flex-row-reverse' : ''}`}>
              {/* Timeline Dot */}
              <div className="absolute left-1/2 top-0 transform -translate-x-1/2 w-4 h-4 rounded-full bg-amber-500 shadow-xl z-10 hidden md:block border-2 border-neutral-800"></div>
              
              {/* Content Card */}
              <div className={`w-full md:w-1/2 p-4 sm:p-6 rounded-2xl shadow-lg transition-all duration-300 hover:shadow-amber-500/30 border border-gray-700 bg-neutral-900 ${index % 2 === 0 ? 'md:pr-12 text-left md:text-right' : 'md:pl-12 text-left'}`}>
                <h4 className="text-xl font-bold text-white mb-1">{exp.title}</h4>
                <p className="text-amber-500 font-semibold">{exp.company}</p>
                <p className="text-sm text-gray-400 mt-1 mb-3">{exp.duration}</p>
                <p className="text-gray-300">{exp.description}</p>
              </div>
              {/* Spacer for non-mobile views */}
              <div className="hidden md:block w-1/2"></div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );

  const CaseStudiesPage = () => (
    <div className="container mx-auto px-4 py-16 pt-32">
      <style>{`
        /* Minimal floating animation for subtle effect */
        @keyframes float {
          0% { transform: translateY(0px) rotate(0deg); }
          50% { transform: translateY(-8px) rotate(1deg); }
          100% { transform: translateY(0px) rotate(0deg); }
        }
        .float-animation {
          animation: float 5s ease-in-out infinite;
        }
      `}</style>
      <h1 className="text-4xl font-extrabold text-center text-white mb-12 border-b-4 border-amber-500 pb-3 inline-block mx-auto">
        Key Case Studies
      </h1>
      <p className="text-center text-gray-400 max-w-2xl mx-auto mb-10">
        Click on any bubble to read the full context of the project, challenges, and successful outcomes.
      </p>
      <div className="flex flex-wrap justify-center gap-8 md:gap-12">
        {userData.caseStudies?.map((study, index) => (
          <div
            key={index}
            className="w-40 h-40 sm:w-48 sm:h-48 bg-neutral-800 rounded-full shadow-lg flex items-center justify-center p-4 text-center cursor-pointer transition-all duration-300 hover:scale-110 hover:shadow-amber-500/50 border border-gray-700 float-animation"
            onClick={() => openModal({ ...study, type: 'caseStudy' })}
            style={{ animationDelay: `${index * 0.5}s` }}
          >
            <div className="flex flex-col items-center">
              <TechLogo name={study.logo} className="w-12 h-12 text-amber-500 mb-2" />
              <span className="font-bold text-lg text-white line-clamp-2">{study.title}</span>
            </div>
          </div>
        ))}
      </div>
    </div>
  );

  const RecommendationsPage = () => (
    <div className="container mx-auto px-4 py-16 pt-32">
       <style>{`
        /* Minimal floating animation for subtle effect */
        @keyframes float {
          0% { transform: translateY(0px) rotate(0deg); }
          50% { transform: translateY(-8px) rotate(1deg); }
          100% { transform: translateY(0px) rotate(0deg); }
        }
        .float-animation {
          animation: float 5s ease-in-out infinite;
        }
      `}</style>
      <h1 className="text-4xl font-extrabold text-center text-white mb-12 border-b-4 border-amber-500 pb-3 inline-block mx-auto">
        Client & Colleague Endorsements
      </h1>
      <p className="text-center text-gray-400 max-w-2xl mx-auto mb-10">
        Hear from those I've worked with. Click on any profile to read the full testimonial.
      </p>
      <div className="flex flex-wrap justify-center gap-8 md:gap-12">
        {userData.recommendations?.map((rec, index) => (
          <div
            key={index}
            className="w-40 h-40 sm:w-48 sm:h-48 bg-neutral-800 rounded-full shadow-lg flex items-center justify-center p-4 text-center cursor-pointer transition-all duration-300 hover:scale-110 hover:shadow-amber-500/50 border border-gray-700 float-animation"
            onClick={() => openModal({ ...rec, type: 'recommendation' })}
            style={{ animationDelay: `${index * 0.5}s` }}
          >
            <div className="flex flex-col items-center">
              <Users className="w-12 h-12 text-amber-500 mb-2" />
              <span className="font-bold text-lg text-white line-clamp-2">{rec.recommender}</span>
              <span className="text-xs text-gray-400 line-clamp-1">{rec.title}</span>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
  
  const ContactPage = () => (
    <div className="container mx-auto px-4 py-16 pt-32">
      <h1 className="text-4xl font-extrabold text-center text-white mb-12 border-b-4 border-amber-500 pb-3 inline-block mx-auto">
        Let's Connect
      </h1>
      <div className="max-w-xl mx-auto bg-neutral-800 rounded-3xl shadow-xl p-8 md:p-12 border-t-4 border-amber-600">
        <p className="text-center text-gray-300 mb-8">
          I'm always open to discussing new opportunities, collaborations, and project inquiries. Please feel free to reach out.
        </p>
        <form className="space-y-6" onSubmit={(e) => { e.preventDefault(); alert("Form submission simulated. Jane Doe will be in touch!"); }}>
          <div>
            <label htmlFor="name" className="block text-sm font-medium text-gray-300">
              Name
            </label>
            <input
              type="text"
              id="name"
              name="name"
              className="mt-1 block w-full rounded-xl border-gray-700 shadow-sm p-3 bg-neutral-900 text-white placeholder-gray-500 focus:border-amber-500 focus:ring-amber-500"
              placeholder="Your Name"
              required
            />
          </div>
          <div>
            <label htmlFor="email" className="block text-sm font-medium text-gray-300">
              Email
            </label>
            <input
              type="email"
              id="email"
              name="email"
              className="mt-1 block w-full rounded-xl border-gray-700 shadow-sm p-3 bg-neutral-900 text-white placeholder-gray-500 focus:border-amber-500 focus:ring-amber-500"
              placeholder="you@example.com"
              required
            />
          </div>
          <div>
            <label htmlFor="message" className="block text-sm font-medium text-gray-300">
              Message
            </label>
            <textarea
              id="message"
              name="message"
              rows="4"
              className="mt-1 block w-full rounded-2xl border-gray-700 shadow-sm p-3 bg-neutral-900 text-white placeholder-gray-500 focus:border-amber-500 focus:ring-amber-500"
              placeholder="Your message to me..."
              required
            ></textarea>
          </div>
          <div className="flex justify-center">
            <button
              type="submit"
              className="w-full md:w-2/3 flex items-center justify-center px-8 py-3 border border-transparent text-base font-medium rounded-full text-neutral-950 bg-amber-500 hover:bg-amber-600 transition-colors shadow-lg shadow-amber-500/30 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-amber-500"
            >
              <Mail className='w-5 h-5 mr-2' />
              Send Message
            </button>
          </div>
        </form>
      </div>
    </div>
  );

  return (
    <div className="bg-neutral-900 min-h-screen text-white font-sans">
      <NavBar />
      <main className="min-h-screen pt-16">
        {(() => {
          switch (currentPage) {
            case 'home':
              return <HomePage />;
            case 'caseStudies':
              return <CaseStudiesPage />;
            case 'recommendations':
              return <RecommendationsPage />;
            case 'contact':
              return <ContactPage />;
            default:
              return <HomePage />;
          }
        })()}
      </main>
      <Modal isOpen={isModalOpen} content={modalContent} onClose={closeModal} />
      {/* Footer */}
      <footer className="w-full text-center py-6 text-gray-500 text-sm border-t border-gray-800 mt-12">
          Portfolio App | Powered by Firebase & React | User ID: {userId}
      </footer>
    </div>
  );
};

export default App;

