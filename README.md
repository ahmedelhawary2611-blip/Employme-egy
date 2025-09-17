import React, { useState } from 'react';
import type { Page, Job, User, Application, Candidate, CandidateStatus, CartItem, AccountType, BillingDetails, BillingCycle, SubscriptionPlanName, NavLink, FooterInfo } from './types';
import Header from './components/Header';
import Footer from './components/Footer';
import HomePage from './components/HomePage';
import AboutPage from './components/AboutPage';
import ServicesPage from './components/ServicesPage';
import PricingPage from './components/PricingPage';
import ContactPage from './components/ContactPage';
import LoginPage from './components/LoginPage';
import SignUpPage from './components/SignUpPage';
import VacanciesPage from './components/VacanciesPage';
import ApplicationPage from './components/ApplicationPage';
import AccountPage from './components/AccountPage';
import AdminDashboardPage from './components/AdminDashboardPage';
import SubscriptionSelectionPage from './components/SubscriptionSelectionPage';
import CartPage from './components/CartPage';
import CheckoutPage from './components/CheckoutPage';
import AdminUserViewPage from './components/AdminUserViewPage';
import ForgotPasswordPage from './components/ForgotPasswordPage';
import { JOBS, PRICING_PLANS, NAV_LINKS } from './constants';
import { GoogleGenAI, Type } from "@google/genai";

const DUMMY_USERS: User[] = [
    { 
        name: 'John Doe', 
        email: 'john.doe@example.com', 
        accountType: 'jobseeker', 
        photo: null,
        linkedinUrl: 'https://linkedin.com/in/johndoe',
        githubUrl: 'https://github.com/johndoe',
        creationDate: '2023-01-15T10:00:00Z',
    },
    { 
        name: 'Jane Smith', 
        email: 'jane.smith@example.com', 
        accountType: 'employer', 
        photo: 'https://www.employme-egy.com/wp-content/uploads/2023/07/spinneys.png', 
        companyName: 'Spinneys',
        subscriptionPlan: 'Small Companies',
        subscriptionCycle: 'monthly',
        subscriptionEndDate: '2024-12-31',
        creationDate: '2023-02-20T11:30:00Z',
    },
    {
        name: 'El Araby Group',
        email: 'hr@elarabygroup.com',
        accountType: 'employer',
        photo: 'https://www.employme-egy.com/wp-content/uploads/2023/07/elaraby.png',
        companyName: 'El Araby Group',
        subscriptionPlan: 'Corporates',
        subscriptionCycle: 'annually',
        subscriptionEndDate: '2025-06-30',
        creationDate: '2023-03-10T09:00:00Z',
    },
    {
        name: 'Waffarha',
        email: 'jobs@waffarha.com',
        accountType: 'employer',
        photo: 'https://www.employme-egy.com/wp-content/uploads/2023/07/waffarha.png',
        companyName: 'Waffarha',
        subscriptionPlan: 'Medium Companies',
        subscriptionCycle: 'semi_annually',
        subscriptionEndDate: '2024-11-30',
        creationDate: '2023-04-05T14:00:00Z',
    },
    {
        name: 'B.TECH',
        email: 'careers@btech.com',
        accountType: 'employer',
        photo: 'https://www.employme-egy.com/wp-content/uploads/2023/07/b-tech.png',
        companyName: 'B.TECH',
        subscriptionPlan: 'Premium',
        subscriptionCycle: 'monthly',
        subscriptionEndDate: '2024-07-31',
        creationDate: '2023-05-12T16:45:00Z',
    },
    {
        name: 'Jotun',
        email: 'recruitment@jotun.com',
        accountType: 'employer',
        photo: 'https://www.employme-egy.com/wp-content/uploads/2023/07/jotun.png',
        companyName: 'Jotun',
        subscriptionPlan: 'Corporates',
        subscriptionCycle: 'quarterly',
        subscriptionEndDate: '2024-09-30',
        creationDate: '2023-06-22T08:20:00Z',
    },
    {
        name: 'Carrefour',
        email: 'apply@maf.ae',
        accountType: 'employer',
        photo: 'https://www.employme-egy.com/wp-content/uploads/2023/07/carrefour.png',
        companyName: 'Carrefour',
        subscriptionPlan: 'Premium',
        subscriptionCycle: 'annually',
        subscriptionEndDate: '2025-01-31',
        creationDate: '2023-07-01T12:00:00Z',
    },
    {
        name: 'Inactive Company',
        email: 'contact@inactive.com',
        accountType: 'employer',
        photo: null,
        companyName: 'Inactive Company',
        creationDate: new Date(Date.now() - 48 * 3600 * 1000).toISOString(), // 48 hours ago
    },
    {
        name: 'Pending Corp',
        email: 'hr@pending.com',
        accountType: 'employer',
        photo: null,
        companyName: 'Pending Corp',
        creationDate: new Date(Date.now() - 12 * 3600 * 1000).toISOString(), // 12 hours ago
    }
];

const ADMIN_USER: User = {
    name: 'Ahmed Elhawary',
    email: 'ahmed_elhawary@employme-egy.com',
    accountType: 'admin',
    photo: null,
    creationDate: '2023-01-01T00:00:00Z',
};
const ADMIN_PASSWORD = 'Employme@2026';

const App: React.FC = () => {
  const [currentPage, setCurrentPage] = useState<Page>('home');
  const [pageParams, setPageParams] = useState<any>(null);
  const [selectedJob, setSelectedJob] = useState<Job | null>(null);
  const [currentUser, setCurrentUser] = useState<User | null>(null);
  const [userApplications, setUserApplications] = useState<Application[]>([]);
  const [userPostings, setUserPostings] = useState<Job[]>([]);
  const [allJobs, setAllJobs] = useState<Job[]>(JOBS);
  const [candidates, setCandidates] = useState<Candidate[]>([]);
  const [allUsers, setAllUsers] = useState<User[]>([...DUMMY_USERS, ADMIN_USER]);
  
  // New state for checkout flow
  const [cartItem, setCartItem] = useState<CartItem | null>(null);
  const [billingDetails, setBillingDetails] = useState<BillingDetails | null>(null);
  const [postLoginRedirect, setPostLoginRedirect] = useState<{page: Page, params: any} | null>(null);
  
  // Site content state
  const [companyLogo, setCompanyLogo] = useState<string | null>('https://www.employme-egy.com/wp-content/uploads/2023/07/logo.png');
  const [navLinks, setNavLinks] = useState<NavLink[]>(NAV_LINKS);
  const [footerInfo, setFooterInfo] = useState<FooterInfo>({
      contactEmail: 'info@employme-egy.com',
      contactPhone: '+201114777082',
      taxRegistrar: '633-814-318',
      commercialRecord: '101799',
      facebookUrl: 'https://web.facebook.com/EmploymeEgypt/',
      linkedinUrl: 'https://www.linkedin.com/company/42942367',
      instagramUrl: 'https://www.instagram.com/employme_egypt/',
  });

  const clientLogos = allUsers.filter(u => u.accountType === 'employer' && u.photo).map(u => u.photo as string);

  const navigate = (page: Page, params?: any) => {
    setPageParams(params);
    setCurrentPage(page);
    window.scrollTo(0, 0);
  };

  const navigateToApplication = (job: Job) => {
    setSelectedJob(job);
    navigate('application');
  };

  const handleSignUp = (user: User) => {
    const newUser: User = {
      ...user,
      creationDate: new Date().toISOString(),
      dateOfBirth: '1990-01-01',
      phone: '+201234567890',
      companyName: user.accountType === 'employer' ? 'My Company' : undefined,
    };
    setCurrentUser(newUser);
    setAllUsers(prev => [...prev, newUser]);
    setUserApplications([]);
    setUserPostings([]);
    
    if (newUser.accountType === 'employer') {
        // Redirect new employers to choose a subscription plan
        navigate('subscriptionSelection');
    } else {
        navigate('account');
    }
  };

   const handleLogin = (email: string, pass: string) => {
    if (email.toLowerCase() === ADMIN_USER.email && pass === ADMIN_PASSWORD) {
      setCurrentUser(ADMIN_USER);
      navigate('admin');
      return;
    }
    const foundUser = allUsers.find(u => u.email === email);
    if (foundUser) {
        setCurrentUser(foundUser);
        if (postLoginRedirect) {
            navigate(postLoginRedirect.page, postLoginRedirect.params);
            setPostLoginRedirect(null);
        } else {
            navigate('account');
        }
    } else {
        alert('User not found or password incorrect. Please sign up or try again.');
    }
  };

  const handleLogout = () => {
    setCurrentUser(null);
    setUserApplications([]);
    setUserPostings([]);
    setCartItem(null);
    setBillingDetails(null);
    navigate('home');
  };

  const handleApply = async (job: Job, applicantDetails: { name: string; email: string; cvDataUrl: string }): Promise<void> => {
    if (!userApplications.some(app => app.job.id === job.id)) {
        setUserApplications(prev => [...prev, { job, status: 'Submitted' }]);
    }

    let newStatus: CandidateStatus = 'Pending Review';
    let rejectionReason: string | undefined = undefined;

    const [header, base64Data] = applicantDetails.cvDataUrl.split(',');
    const mimeType = header ? header.split(':')[1].split(';')[0] : '';
    
    const supportedMimeTypes = [
        'application/pdf',
        'application/msword',
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
    ];

    // Attempt analysis for supported document types (PDF, DOC, DOCX)
    if (supportedMimeTypes.includes(mimeType) && base64Data) {
        try {
            const ai = new GoogleGenAI({ apiKey: process.env.API_KEY });
            
            const cvPart = {
                inlineData: {
                    mimeType: mimeType,
                    data: base64Data,
                },
            };

            const prompt = `
                You are an expert HR screening assistant. Your task is to analyze the provided CV against the job requirements and determine if the candidate is a suitable match.

                **Job Title:** ${job.title}

                **Job Requirements:**
                - ${job.requirements.join('\n- ')}

                **Analysis Criteria:**
                1.  **Education:** Does the candidate's educational background align with the job requirements?
                2.  **Experience:** Does the candidate have the required years and type of experience?
                3.  **Skills:** Does the candidate possess the key skills mentioned in the requirements?

                Based on a strict evaluation of the CV against these criteria, is the candidate a strong match? A strong match means they meet most, if not all, of the key requirements.

                Respond ONLY with a JSON object matching the provided schema.
            `;

            const textPart = { text: prompt };

            const response = await ai.models.generateContent({
                model: 'gemini-2.5-flash',
                contents: { parts: [cvPart, textPart] },
                config: {
                    responseMimeType: "application/json",
                    responseSchema: {
                        type: Type.OBJECT,
                        properties: {
                            isMatch: {
                                type: Type.BOOLEAN,
                                description: 'True if the candidate is a strong match for the job requirements, false otherwise.'
                            },
                            reason: {
                                type: Type.STRING,
                                description: 'A brief, one-sentence explanation for the decision.'
                            }
                        }
                    }
                }
            });
            
            const resultText = response.text.trim();
            const result = JSON.parse(resultText);
            
            newStatus = result.isMatch ? 'Shortlisted' : 'Rejected';
            if (!result.isMatch) {
                rejectionReason = result.reason;
            }
            
        } catch (error) {
            console.error("CV Analysis Error:", error);
            // If analysis fails for any reason, fall back to manual review
            newStatus = 'Pending Review';
        }
    } else {
        // If MIME type is not supported (or data is invalid), skip analysis and go to manual review
        console.log(`Skipping CV analysis for unsupported MIME type or invalid data: ${mimeType}`);
        newStatus = 'Pending Review';
    }


    const newCandidate: Candidate = {
        id: Date.now(),
        jobId: job.id,
        name: applicantDetails.name,
        email: applicantDetails.email,
        cvDataUrl: applicantDetails.cvDataUrl,
        status: newStatus,
        rejectionReason: rejectionReason,
    };
    setCandidates(prev => [newCandidate, ...prev]);
  };
  
  // Checkout Flow Handlers
  const handleSelectPlan = (planName: SubscriptionPlanName) => {
      if (!currentUser) {
          setPostLoginRedirect({ page: 'subscriptionSelection', params: { selectedPlanName: planName }});
          navigate('login');
          return;
      }
      navigate('subscriptionSelection', { selectedPlanName: planName });
  };

  const handleProceedToCart = (item: CartItem) => {
    setCartItem(item);
    navigate('cart');
  };
  
  const handleProceedToCheckout = (details: BillingDetails) => {
    setBillingDetails(details);
    navigate('checkout');
  };

  const handleSubscription = () => {
    if (!currentUser || currentUser.accountType !== 'employer' || !cartItem || !billingDetails) return;
    
    const endDate = new Date();
    const monthsToAdd = { monthly: 1, quarterly: 3, semi_annually: 6, annually: 12 };
    endDate.setMonth(endDate.getMonth() + monthsToAdd[cartItem.cycle]);

    const updatedUser: User = {
        ...currentUser,
        subscriptionPlan: cartItem.plan.name,
        subscriptionCycle: cartItem.cycle,
        subscriptionEndDate: endDate.toISOString().split('T')[0],
        companyName: billingDetails.company,
        phone: billingDetails.phone,
        billingDetails: billingDetails,
    };
    setCurrentUser(updatedUser);
    setAllUsers(prev => prev.map(u => u.email === updatedUser.email ? updatedUser : u));
    setCartItem(null);
    setBillingDetails(null);
  };


  // Job and User Management
  const handleAddJob = (newJobData: Omit<Job, 'id' | 'status'>) => {
    const newJob: Job = { ...newJobData, id: Date.now(), status: 'open' };
    setUserPostings(prev => [newJob, ...prev]);
    setAllJobs(prev => [newJob, ...prev]);
  };
  
  const handleUpdateJob = (updatedJob: Job) => {
      setUserPostings(prev => prev.map(j => j.id === updatedJob.id ? updatedJob : j));
      setAllJobs(prev => prev.map(j => j.id === updatedJob.id ? updatedJob : j));
  };

  const handleDeleteJob = (jobId: number) => {
      if (window.confirm("Are you sure you want to delete this job posting? This action cannot be undone.")) {
          setUserPostings(prev => prev.filter(j => j.id !== jobId));
          setAllJobs(prev => prev.filter(j => j.id !== jobId));
      }
  };

  const handleRepostJob = (job: Job) => {
      const newJob: Job = { ...job, id: Date.now(), status: 'open' };
      setUserPostings(prev => [newJob, ...prev]);
      setAllJobs(prev => [newJob, ...prev]);
  };

  const handleUpdateUser = (updatedUser: Partial<User>) => {
    if (currentUser) {
        const newCurrentUser = { ...currentUser, ...updatedUser };
        setCurrentUser(newCurrentUser);
        setAllUsers(prevUsers => prevUsers.map(u => u.email === newCurrentUser.email ? newCurrentUser : u));
    }
  };
  
  const handleUpdateCandidateStatus = (candidateId: number, status: CandidateStatus) => {
    setCandidates(prev => prev.map(c => c.id === candidateId ? { ...c, status } : c));
  };
  
  const handleUpdateSiteSettings = (settings: { logo?: string | null; navLinks?: NavLink[]; footerInfo?: FooterInfo }) => {
    if (settings.logo !== undefined) setCompanyLogo(settings.logo);
    if (settings.navLinks) setNavLinks(settings.navLinks);
    if (settings.footerInfo) setFooterInfo(settings.footerInfo);
  };

  const renderPage = () => {
    switch (currentPage) {
      case 'home':
        return <HomePage navigate={navigate} clientLogos={clientLogos} />;
      case 'about':
        return <AboutPage />;
      case 'services':
        return <ServicesPage navigate={navigate} />;
      case 'pricing':
        return <PricingPage user={currentUser} onSelectPlan={handleSelectPlan} navigate={navigate} />;
      case 'contact':
        return <ContactPage />;
      case 'login':
        return <LoginPage navigate={navigate} onLogin={handleLogin} />;
      case 'forgotPassword':
        return <ForgotPasswordPage navigate={navigate} />;
      case 'signup':
        return <SignUpPage navigate={navigate} onSignUp={handleSignUp} preselectedType={pageParams?.preselectedType as AccountType | undefined} />;
      case 'vacancies':
        return <VacanciesPage jobs={allJobs} navigateToApplication={navigateToApplication} />;
      case 'application':
        return <ApplicationPage job={selectedJob} navigate={navigate} onApply={handleApply} />;
      case 'subscriptionSelection':
        return <SubscriptionSelectionPage plans={PRICING_PLANS} onSelect={handleProceedToCart} selectedPlanName={pageParams?.selectedPlanName}/>;
      case 'cart':
        return <CartPage cartItem={cartItem} user={currentUser} onProceed={handleProceedToCheckout} navigate={navigate} />;
      case 'checkout':
        return <CheckoutPage cartItem={cartItem} billingDetails={billingDetails} onConfirm={handleSubscription} navigate={navigate} />;
      case 'account':
        return <AccountPage 
                    user={currentUser} 
                    applications={userApplications} 
                    postings={userPostings} 
                    candidates={candidates}
                    onAddJob={handleAddJob}
                    onUpdateJob={handleUpdateJob}
                    onDeleteJob={handleDeleteJob}
                    onRepostJob={handleRepostJob}
                    navigate={navigate} 
                    onLogout={handleLogout} 
                    onUpdateUser={handleUpdateUser}
                />;
       case 'admin':
        return currentUser?.accountType === 'admin' ? 
            <AdminDashboardPage 
                users={allUsers}
                jobs={allJobs}
                candidates={candidates}
                onDeleteJob={handleDeleteJob}
                onUpdateCandidateStatus={handleUpdateCandidateStatus}
                navigate={navigate}
                companyLogo={companyLogo}
                navLinks={navLinks}
                footerInfo={footerInfo}
                onUpdateSiteSettings={handleUpdateSiteSettings}
            /> : <HomePage navigate={navigate} clientLogos={clientLogos} />;
      case 'adminUserView':
        return currentUser?.accountType === 'admin' ?
            <AdminUserViewPage
                userToView={pageParams?.user}
                allJobs={allJobs}
                candidates={candidates}
                navigate={navigate}
            /> : <HomePage navigate={navigate} clientLogos={clientLogos} />;
      default:
        return <HomePage navigate={navigate} clientLogos={clientLogos} />;
    }
  };

  return (
    <div className="flex flex-col min-h-screen font-sans text-gray-800 bg-white">
      <Header currentPage={currentPage} navigate={navigate} user={currentUser} onLogout={handleLogout} companyLogo={companyLogo} navLinks={navLinks} />
      <main className="flex-grow">
        {renderPage()}
      </main>
      <Footer navigate={navigate} companyLogo={companyLogo} navLinks={navLinks} footerInfo={footerInfo} />
    </div>
  );
};

export default App;
