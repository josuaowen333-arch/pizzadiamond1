import React, { useState, useEffect } from 'react';
import { Camera, Upload, Phone, ChevronLeft, CheckCircle, Circle, Loader } from 'lucide-react';

const JewelryRepairTracker = () => {
  // State management
  const [screen, setScreen] = useState('login'); // login, tracker, admin
  const [repairs, setRepairs] = useState([
    {
      id: 'demo-5551234567-R2025001',
      phone: '5551234567',
      repairId: 'R-2025-001',
      item: '14K White Gold Wedding Band',
      status: 3,
      estimatedDate: '2025-10-28',
      notes: 'Ring sizing from 7 to 6.5. Minor scratches on band will be polished.'
    },
    {
      id: 'demo-5551234567-R2025002',
      phone: '5551234567',
      repairId: 'R-2025-002',
      item: 'Sterling Silver Chain Necklace',
      status: 2,
      estimatedDate: '2025-10-25',
      notes: 'Chain extension from 18" to 20". Good condition, minimal tarnishing.'
    }
  ]);
  const [currentPhone, setCurrentPhone] = useState('');
  const [phoneInput, setPhoneInput] = useState('');
  const [adminPassword, setAdminPassword] = useState('');
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [uploadStatus, setUploadStatus] = useState('');
  const [companyInfo, setCompanyInfo] = useState({
    name: 'Your Jewelry Store',
    tagline: 'Fine Jewelry & Expert Repairs',
    logo: null,
    phone: '+1 (555) 123-4567'
  });

  // Status stages configuration
  const statusStages = [
    { id: 1, name: "Received", description: "We have your jewelry safe in our store" },
    { id: 2, name: "Being Assessed", description: "Our experts are evaluating your piece" },
    { id: 3, name: "Repair in Progress", description: "Your jewelry is being carefully repaired" },
    { id: 4, name: "Quality Inspection", description: "Final quality check in progress" },
    { id: 5, name: "Ready for Pickup", description: "Your jewelry is ready for pickup" }
  ];

  // Load data from localStorage on mount (simulating persistent storage)
  useEffect(() => {
    const savedRepairs = localStorage.getItem('repairData');
    const savedCompany = localStorage.getItem('companyInfo');
    
    if (savedRepairs) {
      try {
        setRepairs(JSON.parse(savedRepairs));
      } catch (e) {
        console.error('Error loading saved repairs:', e);
      }
    }
    
    if (savedCompany) {
      try {
        setCompanyInfo(JSON.parse(savedCompany));
      } catch (e) {
        console.error('Error loading company info:', e);
      }
    }
  }, []);

  // Helper functions
  const cleanPhone = (phone) => {
    return phone.replace(/\D/g, '');
  };

  const formatPhoneDisplay = (phone) => {
    const cleaned = cleanPhone(phone);
    if (cleaned.length === 10) {
      return `(${cleaned.slice(0,3)}) ${cleaned.slice(3,6)}-${cleaned.slice(6)}`;
    }
    return phone;
  };

  const validatePhone = (phone) => {
    const cleaned = cleanPhone(phone);
    return cleaned.length >= 10;
  };

  const formatDate = (dateString) => {
    try {
      const date = new Date(dateString);
      return date.toLocaleDateString('en-US', { 
        weekday: 'long', 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric' 
      });
    } catch {
      return dateString;
    }
  };

  // CSV parsing with validation
  const parseCSV = (csvText) => {
    const lines = csvText.trim().split('\n');
    if (lines.length < 2) {
      throw new Error('CSV file is empty or has no data rows');
    }

    const headers = lines[0].split(',').map(h => h.trim().toLowerCase());
    const expectedHeaders = ['phone', 'repairid', 'item', 'status', 'estimateddate'];
    
    const hasAllHeaders = expectedHeaders.every(h => headers.includes(h));
    if (!hasAllHeaders) {
      throw new Error(`CSV must have headers: ${expectedHeaders.join(', ')}`);
    }

    const parsedData = [];
    const errors = [];

    for (let i = 1; i < lines.length; i++) {
      const values = lines[i].split(',').map(v => v.trim());
      
      if (values.length < 5) {
        errors.push(`Row ${i}: Incomplete data`);
        continue;
      }

      const phone = cleanPhone(values[0]);
      const status = parseInt(values[3]);

      if (!phone || phone.length < 10) {
        errors.push(`Row ${i}: Invalid phone number`);
        continue;
      }

      if (isNaN(status) || status < 1 || status > 5) {
        errors.push(`Row ${i}: Invalid status (must be 1-5)`);
        continue;
      }

      parsedData.push({
        id: `${phone}-${values[1]}-${Date.now()}`,
        phone: phone,
        repairId: values[1],
        item: values[2],
        status: status,
        estimatedDate: values[4],
        notes: values[5] || '',
        photos: []
      });
    }

    if (errors.length > 0) {
      console.warn('CSV parsing warnings:', errors);
    }

    return parsedData;
  };

  // Handlers
  const handleLogoUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => {
        const newCompanyInfo = { ...companyInfo, logo: event.target.result };
        setCompanyInfo(newCompanyInfo);
        localStorage.setItem('companyInfo', JSON.stringify(newCompanyInfo));
      };
      reader.readAsDataURL(file);
    }
  };

  const handleCSVUpload = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (event) => {
      try {
        const newRepairs = parseCSV(event.target.result);
        setRepairs(newRepairs);
        localStorage.setItem('repairData', JSON.stringify(newRepairs));
        setUploadStatus(`✓ Success! ${newRepairs.length} repairs loaded.`);
        setTimeout(() => setUploadStatus(''), 3000);
      } catch (error) {
        setUploadStatus(`✗ Error: ${error.message}`);
      }
    };
    reader.readAsText(file);
  };

  const handleAdminLogin = () => {
    // In production, this would be a real API call
    if (adminPassword === 'demo123') {
      setIsAuthenticated(true);
      setScreen('admin');
      setAdminPassword('');
    } else {
      alert('Invalid password. Demo password is "demo123"');
    }
  };

  const handleSearch = () => {
    const cleaned = cleanPhone(phoneInput);
    
    if (!validatePhone(phoneInput)) {
      alert('Please enter a valid 10-digit phone number');
      return;
    }

    const customerRepairs = repairs.filter(r => r.phone === cleaned);
    setCurrentPhone(cleaned);
    setScreen('tracker');
  };

  // Component renders
  const renderLogo = () => (
    <div className="bg-white border-b border-gray-100 p-10 text-center relative">
      <button
        onClick={() => document.getElementById('logoUpload').click()}
        className="absolute top-5 right-5 w-8 h-8 border border-gray-200 rounded-lg flex items-center justify-center text-gray-600 hover:bg-gray-50 transition-colors"
        title="Upload Logo"
      >
        <Upload size={14} />
      </button>
      <input
        id="logoUpload"
        type="file"
        accept="image/*"
        onChange={handleLogoUpload}
        className="hidden"
      />
      
      <div className="w-16 h-16 mx-auto mb-6 bg-gray-900 rounded-xl flex items-center justify-center text-white text-2xl hover:scale-105 transition-transform">
        {companyInfo.logo ? (
          <img src={companyInfo.logo} alt="Logo" className="w-10 h-10 object-contain rounded-lg" />
        ) : (
          '◇'
        )}
      </div>
      
      <h1 className="text-2xl font-semibold text-gray-900 mb-2 tracking-tight">
        {companyInfo.name}
      </h1>
      <p className="text-sm text-gray-600">{companyInfo.tagline}</p>
    </div>
  );

  const renderLogin = () => (
    <div className="p-10">
      <div className="mb-8">
        <label className="block mb-2 text-sm font-medium text-gray-900">
          Phone Number
        </label>
        <input
          type="tel"
          value={phoneInput}
          onChange={(e) => setPhoneInput(e.target.value)}
          placeholder="(555) 123-4567"
          className="w-full px-4 py-4 border border-gray-200 rounded-lg text-base focus:outline-none focus:ring-2 focus:ring-gray-900 focus:border-transparent transition-all"
          onKeyPress={(e) => e.key === 'Enter' && handleSearch()}
        />
      </div>
      
      <button
        onClick={handleSearch}
        className="w-full py-4 bg-gray-900 text-white rounded-lg font-medium hover:bg-gray-800 transition-all hover:-translate-y-0.5 mb-3"
      >
        Track My Repairs
      </button>
      
      <button
        onClick={() => setScreen('adminLogin')}
        className="w-full py-4 bg-white text-gray-900 border border-gray-200 rounded-lg font-medium hover:bg-gray-50 transition-all"
      >
        Admin Access
      </button>
    </div>
  );

  const renderAdminLogin = () => (
    <div className="p-10">
      <button
        onClick={() => setScreen('login')}
        className="w-full py-4 mb-8 bg-white text-gray-900 border border-gray-200 rounded-lg font-medium hover:bg-gray-50 transition-all flex items-center justify-center gap-2"
      >
        <ChevronLeft size={16} /> Back
      </button>
      
      <h2 className="text-xl font-medium text-gray-900 mb-6">Admin Login</h2>
      
      <div className="mb-6">
        <label className="block mb-2 text-sm font-medium text-gray-900">
          Password
        </label>
        <input
          type="password"
          value={adminPassword}
          onChange={(e) => setAdminPassword(e.target.value)}
          placeholder="Enter admin password"
          className="w-full px-4 py-4 border border-gray-200 rounded-lg text-base focus:outline-none focus:ring-2 focus:ring-gray-900 focus:border-transparent"
          onKeyPress={(e) => e.key === 'Enter' && handleAdminLogin()}
        />
        <p className="mt-2 text-xs text-gray-500">Demo password: demo123</p>
      </div>
      
      <button
        onClick={handleAdminLogin}
        className="w-full py-4 bg-gray-900 text-white rounded-lg font-medium hover:bg-gray-800 transition-all"
      >
        Login
      </button>
    </div>
  );

  const renderAdmin = () => (
    <div className="p-10">
      <button
        onClick={() => {
          setScreen('login');
          setIsAuthenticated(false);
        }}
        className="w-full py-4 mb-8 bg-white text-gray-900 border border-gray-200 rounded-lg font-medium hover:bg-gray-50 transition-all flex items-center justify-center gap-2"
      >
        <ChevronLeft size={16} /> Logout
      </button>
      
      <h2 className="text-xl font-medium text-gray-900 mb-6">Upload Repair Data</h2>
      
      <div
        onClick={() => document.getElementById('csvFile').click()}
        className="border-2 border-dashed border-gray-200 rounded-lg p-8 text-center cursor-pointer hover:border-gray-300 hover:bg-gray-50 transition-all mb-6"
      >
        <Upload className="mx-auto mb-3 text-gray-400" size={32} />
        <p className="font-medium text-gray-900 mb-2">Upload CSV File</p>
        <small className="text-sm text-gray-600">Phone, RepairID, Item, Status, EstimatedDate, Notes (optional)</small>
      </div>
      
      <input
        id="csvFile"
        type="file"
        accept=".csv"
        onChange={handleCSVUpload}
        className="hidden"
      />
      
      {uploadStatus && (
        <div className="bg-gray-50 border border-gray-200 rounded-lg p-4 mb-6 text-sm text-gray-900">
          {uploadStatus}
        </div>
      )}
      
      <div className="bg-gray-50 rounded-lg p-5 font-mono text-xs border border-gray-100">
        Phone,RepairID,Item,Status,EstimatedDate,Notes<br/>
        5551234567,R-2025-001,14K Gold Ring,3,2025-10-28,Ring sizing 7 to 6.5<br/>
        5551234567,R-2025-002,Silver Chain,2,2025-10-25,Extension 18" to 20"
      </div>
      
      <div className="mt-5 text-sm text-gray-600 space-y-1">
        <p><strong className="text-gray-900">Status Numbers:</strong></p>
        <p>1 = Received | 2 = Being Assessed | 3 = Repair in Progress</p>
        <p>4 = Quality Inspection | 5 = Ready for Pickup</p>
      </div>
      
      <div className="mt-8 p-4 bg-blue-50 border border-blue-100 rounded-lg text-sm text-blue-900">
        <strong>Demo Data Loaded:</strong> Try phone number (555) 123-4567 to see example repairs
      </div>
      
      <div className="mt-4 p-4 bg-gray-50 border border-gray-100 rounded-lg text-sm text-gray-600">
        <strong className="text-gray-900">Current Repairs:</strong> {repairs.length} items in system
      </div>
    </div>
  );

  const renderRepairItem = (repair) => {
    const customerRepairs = repairs.filter(r => r.phone === currentPhone);
    
    return (
      <div key={repair.id} className="bg-white border border-gray-100 rounded-xl p-6 mb-6 hover:border-gray-200 hover:shadow-md transition-all">
        <div className="flex justify-between items-start mb-6">
          <h3 className="text-lg font-semibold text-gray-900">{repair.item}</h3>
          <span className="bg-gray-100 text-gray-700 text-xs font-medium px-2 py-1 rounded">
            {repair.repairId}
          </span>
        </div>
        
        <div className="space-y-5 mb-6">
          {statusStages.map((stage, index) => {
            const isCompleted = stage.id < repair.status;
            const isCurrent = stage.id === repair.status;
            const isPending = stage.id > repair.status;
            
            return (
              <div key={stage.id} className="flex items-start relative">
                {index < statusStages.length - 1 && (
                  <div className="absolute left-3 top-6 w-0.5 h-7 bg-gray-100" />
                )}
                
                <div className={`w-6 h-6 rounded-full flex items-center justify-center text-xs font-semibold mr-4 mt-0.5 relative z-10 flex-shrink-0 ${
                  isCompleted ? 'bg-gray-900 text-white' :
                  isCurrent ? 'bg-gray-900 text-white animate-pulse' :
                  'bg-gray-100 text-gray-400 border border-gray-200'
                }`}>
                  {isCompleted ? <CheckCircle size={14} /> : 
                   isCurrent ? <Loader size={14} /> :
                   <Circle size={14} />}
                </div>
                
                <div>
                  <h4 className="text-sm font-medium text-gray-900 mb-1">{stage.name}</h4>
                  <p className="text-xs text-gray-600 leading-relaxed">{stage.description}</p>
                </div>
              </div>
            );
          })}
        </div>
        
        <div className="bg-gray-50 rounded-lg p-5 text-center">
          <h4 className="text-sm font-medium text-gray-900 mb-1">Estimated Completion</h4>
          <p className="text-xs text-gray-600">{formatDate(repair.estimatedDate)}</p>
        </div>
        
        {repair.notes && (
          <div className="mt-5 bg-blue-50 border border-blue-100 rounded-lg p-4">
            <h4 className="text-xs font-medium text-blue-900 mb-1">Repair Notes</h4>
            <p className="text-xs text-blue-700 leading-relaxed">{repair.notes}</p>
          </div>
        )}
        
        <button
          onClick={() => alert(`Would call: ${companyInfo.phone}`)}
          className="w-full mt-5 py-3 bg-white text-gray-900 border border-gray-200 rounded-lg font-medium hover:bg-gray-50 transition-all flex items-center justify-center gap-2"
        >
          <Phone size={16} /> Call Store
        </button>
      </div>
    );
  };

  const renderTracker = () => {
    const customerRepairs = repairs.filter(r => r.phone === currentPhone);
    
    return (
      <div className="p-10">
        <button
          onClick={() => {
            setScreen('login');
            setPhoneInput('');
          }}
          className="w-full py-4 mb-8 bg-white text-gray-900 border border-gray-200 rounded-lg font-medium hover:bg-gray-50 transition-all flex items-center justify-center gap-2"
        >
          <ChevronLeft size={16} /> Back to Search
        </button>
        
        {customerRepairs.length === 0 ? (
          <div className="text-center py-16">
            <h3 className="text-lg font-medium text-gray-900 mb-2">No repairs found</h3>
            <p className="text-sm text-gray-600 leading-relaxed">
              We couldn't find any repairs for {formatPhoneDisplay(currentPhone)}.<br/>
              Please check your number or contact the store.
            </p>
          </div>
        ) : (
          customerRepairs.map(repair => renderRepairItem(repair))
        )}
      </div>
    );
  };

  // Main render
  return (
    <div className="min-h-screen bg-gray-50">
      <div className="max-w-md mx-auto bg-white min-h-screen shadow-xl">
        {renderLogo()}
        
        <div className="bg-white border-b border-gray-100 p-8 text-center">
          <h2 className="text-xl font-medium text-gray-900 mb-2">Repair Tracker</h2>
          <p className="text-sm text-gray-600">Track your jewelry repair status</p>
        </div>
        
        {screen === 'login' && renderLogin()}
        {screen === 'adminLogin' && renderAdminLogin()}
        {screen === 'admin' && isAuthenticated && renderAdmin()}
        {screen === 'tracker' && renderTracker()}
      </div>
    </div>
  );
};

export default JewelryRepairTracker;
