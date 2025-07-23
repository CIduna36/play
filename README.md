<!DOCTYPE html>
<html lang="en" class="scroll-smooth">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1"/>
  <title>PlayWith – Streamers</title>
  <script src="https://cdn.tailwindcss.com  "></script>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js  "></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js  "></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js  "></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css  " />
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400  ;500;700&display=swap" rel="stylesheet">
  <style>
    :root {
      --color-primary: #FF7A00;
      --color-cream: #FFF8E7;
      --gradient-primary: linear-gradient(135deg, #FF7A00 0%, #FF9F40 100%);
      --shadow-glow: 0 0 30px rgba(255, 122, 0, 0.3);
    }
    html.dark {
      --color-cream: #1E1E1E;
      --shadow-glow: 0 0 30px rgba(255, 122, 0, 0.2);
    }
    
    .animate-float {
      animation: float 6s ease-in-out infinite;
    }
    
    @keyframes float {
      0%, 100% { transform: translateY(0px); }
      50% { transform: translateY(-10px); }
    }
    
    .animate-pulse-slow {
      animation: pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite;
    }
    
    .gradient-text {
      background: var(--gradient-primary);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
    }
    
    .glass-effect {
      backdrop-filter: blur(20px);
      background: rgba(255, 255, 255, 0.1);
      border: 1px solid rgba(255, 255, 255, 0.2);
    }
    
    .dark .glass-effect {
      background: rgba(0, 0, 0, 0.3);
      border: 1px solid rgba(255, 255, 255, 0.1);
    }
    
    .live-indicator {
      position: relative;
    }
    
    .live-indicator::before {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: #ff4444;
      border-radius: 50%;
      animation: ping 2s cubic-bezier(0, 0, 0.2, 1) infinite;
    }
    
    @keyframes ping {
      75%, 100% { transform: scale(2); opacity: 0; }
    }
  </style>
</head>

<body class="bg-gradient-to-br from-gray-50 to-gray-100 dark:from-gray-900 dark:to-gray-800 text-gray-800 dark:text-gray-100 transition-all duration-500 font-poppins">
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect, createContext, useContext, useRef } = React;

    /* ---------- CONFIG ---------- */
    const CONFIG = {
      streamers: [
        { 
          id: 1, 
          name: 'John "JJ" Martinez', 
          game: 'UFC 5', 
          price: 15, 
          avatar: 'https://i.pravatar.cc/200?u=jj  ', 
          rating: 4.9, 
          reviews: 128,
          isLive: true,
          viewers: 1247,
          specialties: ['MMA', 'Combat Sports', 'Strategy'],
          description: 'Professional UFC player with 5+ years experience. Teaching advanced techniques and having fun!'
        }
      ],
    };

    /* ---------- AUTH CONTEXT ---------- */
    const AuthContext = createContext(null);
    const useAuth = () => useContext(AuthContext);

    const AuthProvider = ({ children }) => {
      const [user, setUser] = useState(null);
      const [isLoading, setIsLoading] = useState(false);
      
      const login = async (email) => {
        setIsLoading(true);
        await new Promise(resolve => setTimeout(resolve, 1000));
        const u = { 
          email, 
          avatar: `https://i.pravatar.cc/200?u=  ${email}`,
          name: email.split('@')[0],
          joinDate: new Date().toISOString().split('T')[0]
        };
        setUser(u);
        setIsLoading(false);
      };
      
      const logout = () => {
        setUser(null);
      };
      
      const register = login;
      
      return (
        <AuthContext.Provider value={{ user, login, logout, register, isLoading }}>
          {children}
        </AuthContext.Provider>
      );
    };

    /* ---------- THEME ---------- */
    const useTheme = () => {
      const [theme, setTheme] = useState('light');
      
      useEffect(() => {
        document.documentElement.classList.toggle('dark', theme === 'dark');
      }, [theme]);
      
      const toggleTheme = () => {
        setTheme(prev => prev === 'light' ? 'dark' : 'light');
      };
      
      return [theme, toggleTheme];
    };

    /* ---------- MODAL ---------- */
    const Modal = ({ open, onClose, title, children }) => {
      useEffect(() => {
        if (open) {
          document.body.style.overflow = 'hidden';
        } else {
          document.body.style.overflow = 'unset';
        }
        return () => {
          document.body.style.overflow = 'unset';
        };
      }, [open]);

      if (!open) return null;
      
      return (
        <div className="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50 px-4 animate-in fade-in duration-200" onClick={onClose}>
          <div className="bg-white dark:bg-gray-800 rounded-2xl shadow-2xl p-6 w-full max-w-md transform animate-in zoom-in duration-200" onClick={e => e.stopPropagation()}>
            <div className="flex justify-between items-center mb-6">
              <h2 className="text-2xl font-bold gradient-text">{title}</h2>
              <button onClick={onClose} className="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300 text-xl">
                <i className="fas fa-times"></i>
              </button>
            </div>
            {children}
          </div>
        </div>
      );
    };

    /* ---------- NAVBAR ---------- */
    const Navbar = ({ toggleTheme, theme }) => {
      const { user, logout } = useAuth();
      const [isScrolled, setIsScrolled] = useState(false);

      useEffect(() => {
        const handleScroll = () => {
          setIsScrolled(window.scrollY > 50);
        };
        window.addEventListener('scroll', handleScroll);
        return () => window.removeEventListener('scroll', handleScroll);
      }, []);

      return (
        <nav className={`fixed top-0 w-full z-40 transition-all duration-300 ${
          isScrolled 
            ? 'glass-effect shadow-lg py-2' 
            : 'bg-transparent py-4'
        }`}>
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div className="flex justify-between items-center h-16">
              <div className="flex items-center space-x-2">
                <div className="w-10 h-10 rounded-xl bg-gradient-to-br from-orange-500 to-red-500 flex items-center justify-center">
                  <i className="fas fa-gamepad text-white text-lg"></i>
                </div>
                <span className="text-2xl font-bold gradient-text">PlayWith</span>
              </div>
              
              <div className="flex items-center space-x-6">
                <a href="#hero" className="hover:text-orange-500 transition-colors font-medium">Home</a>
                <a href="#streamers" className="hover:text-orange-500 transition-colors font-medium">Streamers</a>
                
                {user ? (
                  <div className="flex items-center space-x-3">
                    <div className="flex items-center space-x-2 bg-white/10 dark:bg-black/20 rounded-full px-3 py-1">
                      <img src={user.avatar} alt="avatar" className="w-8 h-8 rounded-full border-2 border-orange-500"/>
                      <span className="text-sm font-medium">{user.name}</span>
                    </div>
                    <button 
                      onClick={logout} 
                      className="text-sm text-gray-600 dark:text-gray-400 hover:text-orange-500 transition-colors"
                    >
                      <i className="fas fa-sign-out-alt"></i>
                    </button>
                  </div>
                ) : null}
                
                <button 
                  onClick={toggleTheme} 
                  className="p-2 rounded-full hover:bg-white/20 dark:hover:bg-black/20 transition-all duration-300"
                >
                  <i className={`fas ${theme === 'dark' ? 'fa-sun text-yellow-400' : 'fa-moon text-gray-600'} text-lg`}></i>
                </button>
              </div>
            </div>
          </div>
        </nav>
      );
    };

    /* ---------- HERO ---------- */
    const Hero = () => {
      return (
        <section id="hero" className="min-h-screen flex items-center justify-center relative overflow-hidden">
          <div className="absolute inset-0 bg-gradient-to-br from-orange-500 via-red-200 to-orange-200"></div>
          <div className="absolute inset-0 bg-black/20"></div>
          
          <div className="absolute top-20 left-10 w-20 h-20 bg-white/10 rounded-full animate-float"></div>
          <div className="absolute top-40 right-20 w-16 h-16 bg-white/10 rounded-full animate-float" style={{animationDelay: '2s'}}></div>
          <div className="absolute bottom-40 left-1/4 w-12 h-12 bg-white/10 rounded-full animate-float" style={{animationDelay: '4s'}}></div>
          
          <div className="relative z-10 text-center px-4 max-w-4xl mx-auto">
            <div className="mb-8">
              <h1 className="text-6xl md:text-7xl font-bold text-white mb-6 animate-in fade-in slide-in-from-bottom duration-1000 font-poppins">
                Play Games With
                <span className="block bg-gradient-to-r from-yellow-300 to-orange-300 bg-clip-text text-transparent">
                  Pro Streamers
                </span>
              </h1>
              <p className="text-xl md:text-2xl text-white/90 max-w-2xl mx-auto mb-8 animate-in fade-in slide-in-from-bottom duration-1000 delay-300 font-poppins">
                Join live gaming sessions, learn from pros, and have epic gaming experiences while connecting with your favorite streamers!
              </p>
            </div>
            
            <div className="flex flex-col sm:flex-row gap-4 justify-center animate-in fade-in slide-in-from-bottom duration-1000 delay-500">
              <a 
                href="#streamers" 
                className="bg-white text-gray-900 font-bold py-4 px-8 rounded-full shadow-2xl hover:shadow-[var(--shadow-glow)] hover:scale-105 transition-all duration-300 font-poppins"
              >
                <i className="fas fa-play mr-2"></i>
                Browse Streamers
              </a>
              <button className="border-2 border-white text-white font-bold py-4 px-8 rounded-full hover:bg-white hover:text-gray-900 transition-all duration-300 font-poppins">
                <i className="fas fa-info-circle mr-2"></i>
                How it Works
              </button>
            </div>
          </div>
        </section>
      );
    };

    /* ---------- STREAMER CARD ---------- */
    const StreamerCard = ({ streamer }) => {
      const { user } = useAuth();
      const [showLogin, setShowLogin] = useState(false);
      const [showPayment, setShowPayment] = useState(false);
      
      const handleJoin = () => {
        if (user) {
          setShowPayment(true);
        } else {
          setShowLogin(true);
        }
      };

      const handlePaymentSuccess = () => {
        setShowPayment(false);
        alert('🎉 Payment successful! Redirecting to game session...');
      };

      return (
        <>
          <section id="streamers" className="py-20">
            <div className="max-w-6xl mx-auto px-4">
              <div className="text-center mb-12">
                <h2 className="text-4xl font-bold mb-4 gradient-text font-poppins">Featured Streamer</h2>
                <p className="text-gray-600 dark:text-gray-400 text-lg font-poppins">Join the most popular gaming session right now</p>
              </div>
              
              <div className="bg-white/80 dark:bg-gray-800/80 backdrop-blur-lg rounded-3xl shadow-2xl overflow-hidden">
                <div className="p-8 md:p-12">
                  <div className="flex flex-col lg:flex-row items-center gap-8">
                    <div className="relative">
                      <img 
                        src={streamer.avatar} 
                        alt={streamer.name} 
                        className="w-40 h-40 rounded-full object-cover border-4 border-orange-500 shadow-xl"
                      />
                      {streamer.isLive && (
                        <div className="absolute -top-2 -right-2 bg-red-500 text-white px-3 py-1 rounded-full text-sm font-bold flex items-center font-poppins">
                          <div className="w-2 h-2 bg-white rounded-full mr-2 live-indicator"></div>
                          LIVE
                        </div>
                      )}
                    </div>
                    
                    <div className="flex-1 text-center lg:text-left">
                      <h3 className="text-3xl font-bold mb-2 font-poppins">{streamer.name}</h3>
                      <p className="text-gray-600 dark:text-gray-300 mb-2 font-poppins">
                        Playing <span className="font-bold text-orange-600">{streamer.game}</span>
                      </p>
                      <p className="text-gray-500 dark:text-gray-400 mb-4 font-poppins">{streamer.description}</p>
                      
                      <div className="flex flex-wrap justify-center lg:justify-start gap-2 mb-4">
                        {streamer.specialties.map((specialty, index) => (
                          <span key={index} className="bg-orange-100 dark:bg-orange-900 text-orange-800 dark:text-orange-200 px-3 py-1 rounded-full text-sm font-medium font-poppins">
                            {specialty}
                          </span>
                        ))}
                      </div>
                      
                      <div className="flex items-center justify-center lg:justify-start space-x-6 mb-6">
                        <div className="flex items-center">
                          <i className="fas fa-star text-yellow-400 mr-1"></i>
                          <span className="font-bold font-poppins">{streamer.rating}</span>
                          <span className="text-gray-500 ml-1 font-poppins">({streamer.reviews} reviews)</span>
                        </div>
                        {streamer.isLive && (
                          <div className="flex items-center text-red-500">
                            <i className="fas fa-eye mr-1"></i>
                            <span className="font-bold font-poppins">{streamer.viewers.toLocaleString()} watching</span>
                          </div>
                        )}
                      </div>
                      
                      <button
                        onClick={handleJoin}
                        className="bg-gradient-to-r from-orange-500 to-red-500 text-white font-bold py-4 px-8 rounded-full shadow-xl hover:shadow-[var(--shadow-glow)] hover:scale-105 transition-all duration-300 font-poppins"
                      >
                        {user ? (
                          <>
                            <i className="fas fa-gamepad mr-2"></i>
                            Join Game (${streamer.price})
                          </>
                        ) : (
                          <>
                            <i className="fas fa-sign-in-alt mr-2"></i>
                            Login to Join
                          </>
                        )}
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </section>
          
          {showLogin && <LoginModal onClose={() => setShowLogin(false)} />}
          {showPayment && (
            <PaymentModal 
              streamer={streamer} 
              onClose={() => setShowPayment(false)}
              onSuccess={handlePaymentSuccess}
            />
          )}
        </>
      );
    };

    /* ---------- LOGIN / REGISTER MODAL ---------- */
    const LoginModal = ({ onClose }) => {
      const { login, register, isLoading } = useAuth();
      const [isLogin, setIsLogin] = useState(true);
      const [email, setEmail] = useState('');
      const [errors, setErrors] = useState({});
      
      const validateEmail = (email) => {
        const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return re.test(email);
      };
      
      const handleSubmit = async (e) => {
        e.preventDefault();
        const newErrors = {};
        
        if (!email) {
          newErrors.email = 'Email is required';
        } else if (!validateEmail(email)) {
          newErrors.email = 'Please enter a valid email';
        }
        
        if (Object.keys(newErrors).length > 0) {
          setErrors(newErrors);
          return;
        }
        
        setErrors({});
        
        try {
          if (isLogin) {
            await login(email);
          } else {
            await register(email);
          }
          onClose();
        } catch (error) {
          setErrors({ general: 'Something went wrong. Please try again.' });
        }
      };
      
      return (
        <Modal open onClose={onClose} title={isLogin ? 'Welcome Back!' : 'Join PlayWith'}>
          <form onSubmit={handleSubmit} className="space-y-6">
            {errors.general && (
              <div className="bg-red-100 dark:bg-red-900 text-red-700 dark:text-red-300 p-3 rounded-lg text-sm font-poppins">
                {errors.general}
              </div>
            )}
            
            <div>
              <label className="block text-sm font-medium mb-2 font-poppins">Email Address</label>
              <input 
                type="email" 
                value={email} 
                onChange={e => setEmail(e.target.value)} 
                placeholder="Enter your email"
                className={`w-full border rounded-xl px-4 py-3 focus:ring-2 focus:ring-orange-500 focus:border-transparent transition-all ${
                  errors.email ? 'border-red-500' : 'border-gray-300 dark:border-gray-600'
                } dark:bg-gray-700 dark:text-white font-poppins`}
              />
              {errors.email && (
                <p className="text-red-500 text-sm mt-1 font-poppins">{errors.email}</p>
              )}
            </div>
            
            <button 
              type="submit" 
              disabled={isLoading}
              className="w-full bg-gradient-to-r from-orange-500 to-red-500 text-white rounded-xl py-3 font-bold hover:shadow-lg transition-all duration-300 disabled:opacity-50 font-poppins"
            >
              {isLoading ? (
                <>
                  <i className="fas fa-spinner fa-spin mr-2"></i>
                  {isLogin ? 'Logging in...' : 'Creating account...'}
                </>
              ) : (
                <>
                  <i className={`fas ${isLogin ? 'fa-sign-in-alt' : 'fa-user-plus'} mr-2`}></i>
                  {isLogin ? 'Log In' : 'Create Account'}
                </>
              )}
            </button>
            
            <div className="text-center">
              <p className="text-sm text-gray-600 dark:text-gray-400 font-poppins">
                {isLogin ? "Don't have an account?" : 'Already have an account?'}{' '}
                <button 
                  type="button" 
                  onClick={() => setIsLogin(!isLogin)} 
                  className="font-bold text-orange-500 hover:text-orange-600 transition-colors font-poppins"
                >
                  {isLogin ? 'Sign up free' : 'Log in here'}
                </button>
              </p>
            </div>
          </form>
        </Modal>
      );
    };

    /* ---------- PAYMENT MODAL ---------- */
    const PaymentModal = ({ streamer, onClose, onSuccess }) => {
      const { user } = useAuth();
      const [paymentData, setPaymentData] = useState({
        name: user?.name || '',
        amount: streamer.price.toString(),
        message: ''
      });
      const [loading, setLoading] = useState(false);
      const [errors, setErrors] = useState({});

      const handleInputChange = (field, value) => {
        setPaymentData(prev => ({ ...prev, [field]: value }));
        if (errors[field]) {
          setErrors(prev => ({ ...prev, [field]: '' }));
        }
      };

      const validateForm = () => {
        const newErrors = {};
        
        if (!paymentData.name.trim()) newErrors.name = 'Name is required';
        if (parseFloat(paymentData.amount) < 1) newErrors.amount = 'Amount must be at least $1';
        
        return newErrors;
      };

      const handleSubmit = async (e) => {
        e.preventDefault();
        
        const formErrors = validateForm();
        if (Object.keys(formErrors).length > 0) {
          setErrors(formErrors);
          return;
        }
        
        setLoading(true);
        setErrors({});
        
        try {
          await new Promise(resolve => setTimeout(resolve, 2000));
          onSuccess();
        } catch (error) {
          setErrors({ general: 'Payment failed. Please try again.' });
        } finally {
          setLoading(false);
        }
      };

      return (
        <Modal open onClose={onClose} title="Complete Your Payment">
          <div className="mb-6 p-4 bg-orange-50 dark:bg-orange-900/20 rounded-xl">
            <div className="flex items-center space-x-3">
              <img src={streamer.avatar} alt={streamer.name} className="w-12 h-12 rounded-full" />
              <div>
                <h4 className="font-bold font-poppins">{streamer.name}</h4>
                <p className="text-sm text-gray-600 dark:text-gray-400 font-poppins">{streamer.game} Session</p>
              </div>
            </div>
          </div>

          <form onSubmit={handleSubmit} className="space-y-4">
            {errors.general && (
              <div className="bg-red-100 dark:bg-red-900 text-red-700 dark:text-red-300 p-3 rounded-lg text-sm font-poppins">
                {errors.general}
              </div>
            )}

            <div>
              <label className="block text-sm font-medium mb-1 font-poppins">Full Name</label>
              <input 
                value={paymentData.name} 
                onChange={e => handleInputChange('name', e.target.value)} 
                placeholder="John Doe" 
                className={`w-full border rounded-lg px-3 py-2 font-poppins ${errors.name ? 'border-red-500' : 'border-gray-300'} dark:bg-gray-700`}
              />
              {errors.name && <p className="text-red-500 text-sm mt-1 font-poppins">{errors.name}</p>}
            </div>

            <div>
              <label className="block text-sm font-medium mb-1 font-poppins">Amount ($)</label>
              <input 
                type="number" 
                min="1" 
                step="0.01" 
                value={paymentData.amount} 
                onChange={e => handleInputChange('amount', e.target.value)} 
                className={`w-full border rounded-lg px-3 py-2 font-poppins ${errors.amount ? 'border-red-500' : 'border-gray-300'} dark:bg-gray-700`}
              />
              {errors.amount && <p className="text-red-500 text-sm mt-1 font-poppins">{errors.amount}</p>}
            </div>

            <div>
              <label className="block text-sm font-medium mb-1 font-poppins">Message to Streamer (Optional)</label>
              <textarea 
                value={paymentData.message} 
                onChange={e => handleInputChange('message', e.target.value)} 
                placeholder="Add a message for the streamer..." 
                className="w-full border rounded-lg px-3 py-2 font-poppins border-gray-300 dark:bg-gray-700"
                rows="3"
              />
            </div>

            <button 
              type="submit"
              disabled={loading} 
              className="w-full bg-gradient-to-r from-orange-500 to-red-500 text-white rounded-lg py-3 font-bold hover:shadow-lg transition-all duration-300 disabled:opacity-50 font-poppins"
            >
              {loading ? (
                <>
                  <i className="fas fa-spinner fa-spin mr-2"></i>
                  Processing Payment...
                </>
              ) : (
                <>
                  <i className="fas fa-credit-card mr-2"></i>
                  Pay ${paymentData.amount}
                </>
              )}
            </button>
          </form>
        </Modal>
      );
    };

    /* ---------- FOOTER ---------- */
    const Footer = () => (
      <footer className="bg-gradient-to-r from-gray-900 to-gray-800 text-white py-16">
        <div className="max-w-7xl mx-auto px-4">
          <div className="grid grid-cols-1 md:grid-cols-4 gap-8 mb-8">
            <div>
              <div className="flex items-center space-x-2 mb-4">
                <div className="w-8 h-8 rounded-lg bg-gradient-to-br from-orange-500 to-red-500 flex items-center justify-center">
                  <i className="fas fa-gamepad text-white"></i>
                </div>
                <span className="text-xl font-bold font-poppins">PlayWith</span>
              </div>
              <p className="text-gray-400 font-poppins">Connect with pro gamers and level up your gaming experience. Join live sessions and play with your favorite streamers!</p>
            </div>
            <div>
              <h3 className="text-lg font-bold mb-4 font-poppins">Quick Links</h3>
              <ul className="space-y-2 font-poppins">
                <li><a href="#hero" className="hover:text-orange-500 transition-colors">Home</a></li>
                <li><a href="#streamers" className="hover:text-orange-500 transition-colors">Streamers</a></li>
                <li><a href="#contact" className="hover:text-orange-500 transition-colors">Contact</a></li>
              </ul>
            </div>
            <div>
              <h3 className="text-lg font-bold mb-4 font-poppins">Support</h3>
              <ul className="space-y-2 font-poppins">
                <li><a href="#" className="hover:text-orange-500 transition-colors">FAQ</a></li>
                <li><a href="#" className="hover:text-orange-500 transition-colors">Terms of Service</a></li>
                <li><a href="#" className="hover:text-orange-500 transition-colors">Privacy Policy</a></li>
              </ul>
            </div>
            <div>
              <h3 className="text-lg font-bold mb-4 font-poppins">Follow Us</h3>
              <div className="flex space-x-4">
                <a href="https://twitter.com  " className="text-2xl hover:text-orange-500 transition-colors"><i className="fab fa-twitter"></i></a>
                <a href="https://instagram.com  " className="text-2xl hover:text-orange-500 transition-colors"><i className="fab fa-instagram"></i></a>
                <a href="https://discord.com  " className="text-2xl hover:text-orange-500 transition-colors"><i className="fab fa-discord"></i></a>
              </div>
            </div>
          </div>
          <div className="border-t border-gray-700 pt-6 text-center">
            <p className="text-gray-400 font-poppins">© {new Date().getFullYear()} PlayWith. All rights reserved.</p>
          </div>
        </div>
      </footer>
    );

    /* ---------- APP ---------- */
    const App = () => {
      const [theme, toggleTheme] = useTheme();
      const [streamer] = useState(CONFIG.streamers[0]);
      return (
        <AuthProvider>
          <Navbar toggleTheme={toggleTheme} theme={theme}/>
          <Hero/>
          <StreamerCard streamer={streamer}/>
          <Footer/>
        </AuthProvider>
      );
    };

    ReactDOM.render(<App />, document.getElementById('root'));
  </script>
</body>
</html>                                      გააუმჯობესე
