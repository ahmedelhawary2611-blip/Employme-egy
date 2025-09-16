
import React, { useState } from 'react';
import type { Page, Job, User, Application, Candidate, CandidateStatus, CartItem, AccountType, BillingDetails, BillingCycle, SubscriptionPlanName } from './types';
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
import { JOBS, PRICING_PLANS } from './constants';

const DUMMY_USERS: User[] = [
    { name: 'John Doe', email: 'john.doe@example.com', accountType: 'jobseeker', photo: null },
    { 
        name: 'Jane Smith', 
        email: 'jane.smith@example.com', 
        accountType: 'employer', 
        photo: null, 
        companyName: 'Tech Solutions',
        subscriptionPlan: 'Small Companies',
        subscriptionCycle: 'monthly',
        subscriptionEndDate: '2024-12-31'
    },
];

const ADMIN_USER: User = {
    name: 'Ahmed Elhawary',
    email: 'ahmed_elhawary@employme-egy.com',
    accountType: 'admin',
    photo: null,
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

  const handleApply = (job: Job, applicantDetails: { name: string; email: string; cvDataUrl: string }) => {
    if (!userApplications.some(app => app.job.id === job.id)) {
        setUserApplications(prev => [...prev, { job, status: 'Submitted' }]);
    }
    const newCandidate: Candidate = {
        id: Date.now(),
        jobId: job.id,
        name: applicantDetails.name,
        email: applicantDetails.email,
        cvDataUrl: applicantDetails.cvDataUrl,
        status: 'Pending Review',
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
        setCurrentUser(prevUser => ({ ...prevUser!, ...updatedUser }));
    }
  };
  
  const handleUpdateCandidateStatus = (candidateId: number, status: CandidateStatus) => {
    setCandidates(prev => prev.map(c => c.id === candidateId ? { ...c, status } : c));
  };

  const renderPage = () => {
    switch (currentPage) {
      case 'home':
        return <HomePage navigate={navigate} />;
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
            /> : <HomePage navigate={navigate} />;
      case 'adminUserView':
        return currentUser?.accountType === 'admin' ?
            <AdminUserViewPage
                userToView={pageParams?.user}
                allJobs={allJobs}
                candidates={candidates}
                navigate={navigate}
            /> : <HomePage navigate={navigate} />;
      default:
        return <HomePage navigate={navigate} />;
    }
  };

  return (
    <div className="flex flex-col min-h-screen font-sans text-gray-800 bg-white">
      <Header currentPage={currentPage} navigate={navigate} user={currentUser} onLogout={handleLogout} />
      <main className="flex-grow">
        {renderPage()}
      </main>
      <Footer navigate={navigate} />
    </div>
  );
};

export default App;
