import React, { useState, useEffect } from 'react';
import { Settings, Copy, Calculator, Info, ChevronDown, CheckCircle2, ArrowRightLeft, RotateCcw } from 'lucide-react';

// --- Constants & Config ---
const DEFAULT_SETTINGS = {
  commissionRate: 0.25, // percentage
  minCommission: 20.00, // Treated as placeholder/default, can be 0
  applyVatOnPse: true,
};

const BROKER_PRESETS = {
  'Standard/BPI': { commissionRate: 0.25, minCommission: 20.00, applyVatOnPse: true },
  'No Min Comm': { commissionRate: 0.25, minCommission: 0.00, applyVatOnPse: true },
  'FirstMetro': { commissionRate: 0.25, minCommission: 20.00, applyVatOnPse: true },
};

// --- Dealer-Grade Option Lists ---

// A. Order Type (PSE-level)
const PSE_ORDER_TYPES = [
  'Limit', 
  'Market', 
  'Market-to-Limit', 
  'Market on Close', 
  'Limit on Close'
];

// B. Validity (Updated to Object Array to handle Empty Default)
const VALIDITY_TYPES = [
  { label: 'Day (Default)', value: '' }, // Empty string = Day (Suppressed in output)
  { label: 'GTC', value: 'GTC' },
  { label: 'GTD', value: 'GTD' },
  { label: 'IOC', value: 'IOC' },
  { label: 'FOK', value: 'FOK' }
];

// C. Dealer Operational Instructions (Ops Flags)
const DEALER_INSTRUCTIONS = [
  { label: 'None', value: '' },
  { label: 'Amend', value: 'AMEND' },
  { label: 'Iceberg', value: 'ICEBERG' },
  { label: 'Oddlot', value: 'ODDLOT' },
  { label: 'Cross/Block', value: 'CROSS' }
];

// D. Execution Style / Strategy
const EXECUTION_STYLES = [
  { label: 'None', value: '' },
  { label: 'CD (Careful Discretion)', value: 'CD' },
  { label: 'VWAP', value: 'VWAP' },
  { label: 'TWAP', value: 'TWAP' },
  { label: 'POV (Percentage of Vol)', value: 'POV' },
  { label: 'Aggressive', value: 'AGGRESSIVE' },
  { label: 'Passive', value: 'PASSIVE' },
  { label: 'Work Order', value: 'WORK' }
];

// E. Execution Constraints
const EXECUTION_CONSTRAINTS = [
  { label: 'None', value: '' },
  { label: 'OB (Or Better)', value: 'OB' },
];

const INITIAL_INPUTS = {
  code: '',
  side: 'BUY',
  price: '',
  volume: '',
  // Default to most common institutional setup
  orderType: 'Limit',
  validity: '', // Default is empty (implied Day)
  dealerInstr: '', // Ops Flags
  execStyle: '',   // CD/VWAP
  execConstraint: '' // OB
};

// --- Helper Functions ---

const formatCurrency = (val) => {
  return new Intl.NumberFormat('en-PH', {
    style: 'currency',
    currency: 'PHP',
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(val).replace('PHP', '').trim(); 
};

const formatNumber = (val) => {
  return new Intl.NumberFormat('en-US').format(val);
};

const round2 = (num) => {
  return Math.round((num + Number.EPSILON) * 100) / 100;
};

export default function App() {
  // --- State ---
  const [inputs, setInputs] = useState(INITIAL_INPUTS);
  const [settings, setSettings] = useState(DEFAULT_SETTINGS);
  const [showSettings, setShowSettings] = useState(false);
  const [copied, setCopied] = useState(false);
  const [selectedPreset, setSelectedPreset] = useState('Standard/BPI');

  useEffect(() => {
    const savedSettings = localStorage.getItem('pse_calc_settings');
    if (savedSettings) {
      setSettings(JSON.parse(savedSettings));
    }
  }, []);

  useEffect(() => {
    localStorage.setItem('pse_calc_settings', JSON.stringify(settings));
  }, [settings]);

  // --- Logic: Auto-clear OB on Market Orders ---
  useEffect(() => {
    const isMarket = inputs.orderType.toLowerCase().includes('market');
    if (isMarket && inputs.execConstraint === 'OB') {
      setInputs(prev => ({ ...prev, execConstraint: '' }));
    }
  }, [inputs.orderType]);

  // --- Handlers ---

  const handleInputChange = (field, value) => {
    let formattedValue = value;
    if (field === 'code') formattedValue = value.toUpperCase();
    
    setInputs(prev => ({ ...prev, [field]: formattedValue }));
  };

  const handleSettingsChange = (field, value) => {
    const newVal = value === '' ? 0 : value;
    setSettings(prev => ({ ...prev, [field]: newVal }));
    setSelectedPreset('Custom');
  };

  const handleReset = () => {
    setInputs(INITIAL_INPUTS);
    setCopied(false);
  };

  const applyPreset = (presetName) => {
    if (presetName === 'Custom') return;
    setSettings(BROKER_PRESETS[presetName]);
    setSelectedPreset(presetName);
  };

  const copyToClipboard = (text) => {
    const textArea = document.createElement("textarea");
    textArea.value = text;
    document.body.appendChild(textArea);
    textArea.select();
    try {
      document.execCommand('copy');
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    } catch (err) {
      console.error('Fallback copy failed', err);
    }
    document.body.removeChild(textArea);
  };

  // --- Calculation Engine ---

  const calculate = () => {
    const price = parseFloat(inputs.price);
    const volume = parseInt(inputs.volume);

    if (!price || price <= 0 || !volume || volume <= 0) return null;

    const gross = round2(price * volume);
    
    const rawComm = gross * (settings.commissionRate / 100);
    const commission = round2(Math.max(rawComm, settings.minCommission));
    const vatOnComm = round2(commission * 0.12);
    const pseFee = round2(gross * 0.00005);
    const vatOnPse = settings.applyVatOnPse ? round2(pseFee * 0.12) : 0;
    const sccpFee = round2(gross * 0.0001);
    const salesTax = inputs.side === 'SELL' ? round2(gross * 0.001) : 0;

    const totalFees = round2(commission + vatOnComm + pseFee + vatOnPse + sccpFee + salesTax);
    
    let netAmount;
    if (inputs.side === 'BUY') {
      netAmount = round2(gross + totalFees);
    } else {
      netAmount = round2(gross - totalFees);
    }

    const effectivePrice = volume > 0 ? netAmount / volume : 0;

    return {
      gross,
      commission,
      vatOnComm,
      pseFee,
      vatOnPse,
      sccpFee,
      salesTax,
      totalFees,
      netAmount,
      effectivePrice
    };
  };

  const results = calculate();

  // --- Dealer Text Generator ---
  const generateDealerText = () => {
    if (!results) return "Enter details to generate summary.";
    
    const { side, volume, code, price, orderType, validity, dealerInstr, execStyle, execConstraint } = inputs;
    const volStr = formatNumber(volume);
    
    // Rule: Price Token logic
    const isMarket = orderType.toLowerCase().includes('market');
    let priceToken;
    
    if (isMarket) {
      priceToken = "MKT"; 
    } else {
       priceToken = new Intl.NumberFormat('en-US', { 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format(parseFloat(price));
    }
    
    const grossStr = new Intl.NumberFormat('en-US', { 
      minimumFractionDigits: 2, 
      maximumFractionDigits: 2 
    }).format(results.gross);

    // Rule: OB auto-cleared if Market (Handled in Effect, but safe guard here)
    const finalConstraint = isMarket ? "" : execConstraint;

    // Rule: Validity logic (suppress if empty/Day)
    // The validity state is already '' for Day, so we just use it directly.
    // If it's 'IOC' it shows 'IOC'. If '' it shows nothing.
    
    // Construction Order:
    // 1. Act -> SIDE VOL SYMBOL @ PRICE_TOKEN
    // 2. Price Logic -> OB
    // 3. Order Type -> LIMIT / MARKET
    // 4. Validity -> IOC / GTC (only if not Day)
    // 5. Exec Style -> CD / VWAP
    // 6. Ops Flags -> AMEND / CROSS
    
    const parts = [
      side,
      volStr,
      code,
      '@',
      priceToken,
      finalConstraint, 
      orderType.toUpperCase(),
      validity.toUpperCase(),
      execStyle,     
      dealerInstr     
    ];

    // Filter out empty strings and join
    const mainString = parts.filter(p => p && p.trim() !== '').join(' ');

    return `${mainString} (gross PHP ${grossStr}).`;
  };

  const isMarketOrder = inputs.orderType.toLowerCase().includes('market');

  // --- Components ---

  return (
    <div className="min-h-screen bg-slate-100 text-slate-900 font-sans p-4 md:p-8">
      <div className="max-w-6xl mx-auto">
        
        {/* Header */}
        <header className="flex justify-between items-center mb-6">
          <div className="flex items-center gap-3">
            <div className="bg-blue-900 p-2 rounded-lg">
              <Calculator className="w-6 h-6 text-white" />
            </div>
            <div>
              <h1 className="text-2xl font-bold text-slate-800">PSE Order Calc</h1>
              <p className="text-slate-500 text-xs font-medium tracking-wider uppercase">Philippine Equities • Dealer Tool</p>
            </div>
          </div>
          <button 
            onClick={() => setShowSettings(!showSettings)}
            className={`p-2 rounded-md transition-colors ${showSettings ? 'bg-slate-300 text-slate-800' : 'bg-white text-slate-600 hover:bg-slate-50 shadow-sm'}`}
          >
            <Settings className="w-5 h-5" />
          </button>
        </header>

        {/* Config Panel */}
        {showSettings && (
          <div className="mb-6 bg-white rounded-xl shadow-md border border-slate-200 overflow-hidden animate-in fade-in slide-in-from-top-4">
            <div className="p-4 border-b border-slate-100 bg-slate-50 flex justify-between items-center">
              <h3 className="font-semibold text-slate-700">Broker Configuration</h3>
              <div className="flex items-center gap-2">
                <span className="text-sm text-slate-500">Preset:</span>
                <select 
                  value={selectedPreset}
                  onChange={(e) => applyPreset(e.target.value)}
                  className="text-sm border border-slate-300 rounded px-2 py-1 bg-white"
                >
                  <option value="Custom">Custom</option>
                  {Object.keys(BROKER_PRESETS).map(k => <option key={k} value={k}>{k}</option>)}
                </select>
              </div>
            </div>
            <div className="p-6 grid grid-cols-1 md:grid-cols-3 gap-6">
              <div>
                <label className="block text-xs font-bold text-slate-500 uppercase mb-1">Commission Rate (%)</label>
                <input 
                  type="number" 
                  step="0.01"
                  value={settings.commissionRate}
                  onChange={(e) => handleSettingsChange('commissionRate', parseFloat(e.target.value))}
                  className="w-full p-2 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 outline-none"
                />
              </div>
              <div>
                <label className="block text-xs font-bold text-slate-500 uppercase mb-1">Min Commission (PHP)</label>
                <input 
                  type="number" 
                  min="0"
                  value={settings.minCommission}
                  onChange={(e) => handleSettingsChange('minCommission', parseFloat(e.target.value))}
                  className="w-full p-2 border border-slate-300 rounded focus:ring-2 focus:ring-blue-500 outline-none"
                  placeholder="0.00"
                />
                <span className="text-[10px] text-slate-400">Set to 0 if no minimum applies.</span>
              </div>
              <div className="flex items-center gap-3 mt-4 md:mt-0">
                <input 
                  type="checkbox" 
                  id="vatOnPse"
                  checked={settings.applyVatOnPse}
                  onChange={(e) => handleSettingsChange('applyVatOnPse', e.target.checked)}
                  className="w-5 h-5 text-blue-600 rounded focus:ring-blue-500"
                />
                <label htmlFor="vatOnPse" className="text-sm font-medium text-slate-700 select-none cursor-pointer">
                  Apply VAT on PSE Fee
                  <span className="block text-xs text-slate-400 font-normal">Standard practice from 2025</span>
                </label>
              </div>
            </div>
          </div>
        )}

        <div className="grid grid-cols-1 lg:grid-cols-12 gap-6">
          
          {/* Left Column: Input Form */}
          <div className="lg:col-span-5 flex flex-col gap-4">
            <div className="bg-white p-6 rounded-xl shadow-sm border border-slate-200">
              <div className="flex justify-between items-center mb-5">
                <h2 className="text-lg font-semibold text-slate-800 flex items-center gap-2">
                  <ArrowRightLeft className="w-4 h-4 text-slate-400" />
                  Trade Details
                </h2>
                <button 
                  onClick={handleReset}
                  className="flex items-center gap-1.5 text-xs font-medium text-slate-500 hover:text-blue-600 hover:bg-blue-50 px-3 py-1.5 rounded-md transition-all"
                  title="Clear all fields"
                >
                  <RotateCcw className="w-3.5 h-3.5" />
                  Reset
                </button>
              </div>
              
              <div className="space-y-5">
                {/* Side Selection */}
                <div className="grid grid-cols-2 gap-2 p-1 bg-slate-100 rounded-lg">
                  <button
                    onClick={() => handleInputChange('side', 'BUY')}
                    className={`py-2 text-sm font-bold rounded-md transition-all ${inputs.side === 'BUY' ? 'bg-emerald-600 text-white shadow-sm' : 'text-slate-500 hover:text-slate-700'}`}
                  >
                    BUY
                  </button>
                  <button
                    onClick={() => handleInputChange('side', 'SELL')}
                    className={`py-2 text-sm font-bold rounded-md transition-all ${inputs.side === 'SELL' ? 'bg-rose-600 text-white shadow-sm' : 'text-slate-500 hover:text-slate-700'}`}
                  >
                    SELL
                  </button>
                </div>

                {/* Stock Code */}
                <div>
                  <label className="block text-xs font-bold text-slate-500 uppercase mb-1">Stock Code</label>
                  <input 
                    type="text" 
                    value={inputs.code}
                    onChange={(e) => handleInputChange('code', e.target.value)}
                    placeholder="e.g. ALI"
                    className="w-full p-3 bg-slate-50 border border-slate-200 rounded-lg focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none font-mono text-lg font-bold uppercase tracking-wide"
                  />
                </div>

                {/* Price & Volume */}
                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="block text-xs font-bold text-slate-500 uppercase mb-1">
                      {isMarketOrder ? "Ref. Price (Est.)" : "Limit Price (PHP)"}
                    </label>
                    <input 
                      type="number" 
                      value={inputs.price}
                      onChange={(e) => handleInputChange('price', e.target.value)}
                      placeholder="0.00"
                      className="w-full p-3 bg-slate-50 border border-slate-200 rounded-lg focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none font-mono text-lg text-right"
                    />
                  </div>
                  <div>
                    <label className="block text-xs font-bold text-slate-500 uppercase mb-1">Volume</label>
                    <input 
                      type="number" 
                      value={inputs.volume}
                      onChange={(e) => handleInputChange('volume', e.target.value)}
                      placeholder="0"
                      className="w-full p-3 bg-slate-50 border border-slate-200 rounded-lg focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none font-mono text-lg text-right"
                    />
                  </div>
                </div>

                {/* Dealer Grade Instructions Grid */}
                <div className="pt-2 border-t border-slate-100">
                  <label className="block text-xs font-bold text-slate-500 uppercase mb-3 mt-2">Instructions</label>
                  <div className="grid grid-cols-2 gap-4">
                    
                    {/* 1. Order Type */}
                    <div>
                      <label className="block text-[10px] font-semibold text-slate-400 uppercase mb-1">Type</label>
                      <div className="relative">
                        <select 
                          value={inputs.orderType}
                          onChange={(e) => handleInputChange('orderType', e.target.value)}
                          className="w-full p-2.5 text-sm bg-slate-50 border border-slate-200 rounded-lg focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none appearance-none font-medium text-slate-700"
                        >
                          {PSE_ORDER_TYPES.map(t => <option key={t} value={t}>{t}</option>)}
                        </select>
                        <ChevronDown className="absolute right-2 top-3 w-3.5 h-3.5 text-slate-400 pointer-events-none" />
                      </div>
                    </div>

                    {/* 2. Validity */}
                    <div>
                      <label className="block text-[10px] font-semibold text-slate-400 uppercase mb-1">Validity</label>
                      <div className="relative">
                        <select 
                          value={inputs.validity}
                          onChange={(e) => handleInputChange('validity', e.target.value)}
                          className="w-full p-2.5 text-sm bg-slate-50 border border-slate-200 rounded-lg focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none appearance-none font-medium text-slate-700"
                        >
                          {VALIDITY_TYPES.map(opt => <option key={opt.value} value={opt.value}>{opt.label}</option>)}
                        </select>
                        <ChevronDown className="absolute right-2 top-3 w-3.5 h-3.5 text-slate-400 pointer-events-none" />
                      </div>
                    </div>

                    {/* 3. Constraint (OB) */}
                    <div>
                      <label className="block text-[10px] font-semibold text-slate-400 uppercase mb-1">Constraint</label>
                      <div className="relative">
                        <select 
                          value={inputs.execConstraint}
                          onChange={(e) => handleInputChange('execConstraint', e.target.value)}
                          disabled={isMarketOrder}
                          className={`w-full p-2.5 text-sm border border-slate-200 rounded-lg outline-none appearance-none font-medium ${isMarketOrder ? 'bg-slate-100 text-slate-400' : 'bg-slate-50 focus:bg-white focus:ring-2 focus:ring-blue-500 text-slate-700'}`}
                        >
                          {EXECUTION_CONSTRAINTS.map(opt => <option key={opt.value} value={opt.value}>{opt.label}</option>)}
                        </select>
                        <ChevronDown className="absolute right-2 top-3 w-3.5 h-3.5 text-slate-400 pointer-events-none" />
                      </div>
                    </div>

                    {/* 4. Execution Style (CD, VWAP) */}
                    <div>
                      <label className="block text-[10px] font-semibold text-slate-400 uppercase mb-1">Exec Style</label>
                      <div className="relative">
                        <select 
                          value={inputs.execStyle}
                          onChange={(e) => handleInputChange('execStyle', e.target.value)}
                          className="w-full p-2.5 text-sm bg-slate-50 border border-slate-200 rounded-lg focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none appearance-none font-medium text-slate-700"
                        >
                          {EXECUTION_STYLES.map(opt => <option key={opt.value} value={opt.value}>{opt.label}</option>)}
                        </select>
                        <ChevronDown className="absolute right-2 top-3 w-3.5 h-3.5 text-slate-400 pointer-events-none" />
                      </div>
                    </div>

                    {/* 5. Ops Flags (Amend, Iceberg) */}
                    <div className="col-span-2">
                      <label className="block text-[10px] font-semibold text-slate-400 uppercase mb-1">Ops Flags</label>
                      <div className="relative">
                        <select 
                          value={inputs.dealerInstr}
                          onChange={(e) => handleInputChange('dealerInstr', e.target.value)}
                          className="w-full p-2.5 text-sm bg-slate-50 border border-slate-200 rounded-lg focus:bg-white focus:ring-2 focus:ring-blue-500 outline-none appearance-none font-medium text-slate-700"
                        >
                          {DEALER_INSTRUCTIONS.map(opt => <option key={opt.value} value={opt.value}>{opt.label}</option>)}
                        </select>
                        <ChevronDown className="absolute right-2 top-3 w-3.5 h-3.5 text-slate-400 pointer-events-none" />
                      </div>
                    </div>

                  </div>
                </div>

                {/* Error Message */}
                {(!inputs.price || !inputs.volume) && (
                  <div className="flex items-center gap-2 text-amber-600 text-sm bg-amber-50 p-3 rounded-lg">
                    <Info className="w-4 h-4" />
                    <span>Please enter {isMarketOrder ? 'ref' : 'limit'} price and volume.</span>
                  </div>
                )}

              </div>
            </div>
          </div>

          {/* Right Column: Results */}
          <div className="lg:col-span-7 flex flex-col gap-4">
            
            {/* 1. Dealer Text */}
            <div className="bg-white p-0 rounded-xl shadow-sm border border-slate-200 overflow-hidden">
              <div className="p-4 border-b border-slate-100 bg-slate-50 flex justify-between items-center">
                <h3 className="font-semibold text-slate-700">Dealer Order Text</h3>
                <button 
                  onClick={() => results && copyToClipboard(generateDealerText())}
                  disabled={!results}
                  className={`flex items-center gap-2 text-xs font-bold px-3 py-1.5 rounded-md transition-all ${copied ? 'bg-green-100 text-green-700' : 'bg-blue-50 text-blue-600 hover:bg-blue-100'}`}
                >
                  {copied ? <CheckCircle2 className="w-3 h-3" /> : <Copy className="w-3 h-3" />}
                  {copied ? 'COPIED' : 'COPY TEXT'}
                </button>
              </div>
              <div className="p-5 bg-slate-800">
                 <p className="font-mono text-sm leading-relaxed text-slate-200 break-words">
                   {results ? generateDealerText() : <span className="text-slate-500">Waiting for input...</span>}
                 </p>
              </div>
            </div>

            {/* 2. Financial Breakdown */}
            {results && (
              <div className="bg-white p-6 rounded-xl shadow-sm border border-slate-200 animate-in fade-in slide-in-from-bottom-2">
                
                {/* Top Line */}
                <div className="grid grid-cols-3 gap-4 mb-8 text-center">
                  <div className="p-3 bg-slate-50 rounded-lg">
                    <div className="text-xs text-slate-500 font-semibold uppercase tracking-wide mb-1">Gross Value</div>
                    <div className="text-lg font-bold text-slate-800">{formatCurrency(results.gross)}</div>
                  </div>
                  <div className="p-3 bg-slate-50 rounded-lg">
                    <div className="text-xs text-slate-500 font-semibold uppercase tracking-wide mb-1">Total Fees</div>
                    <div className="text-lg font-bold text-rose-600">{formatCurrency(results.totalFees)}</div>
                  </div>
                  <div className={`p-3 rounded-lg ${inputs.side === 'BUY' ? 'bg-emerald-50 border border-emerald-100' : 'bg-blue-50 border border-blue-100'}`}>
                    <div className={`text-xs font-semibold uppercase tracking-wide mb-1 ${inputs.side === 'BUY' ? 'text-emerald-600' : 'text-blue-600'}`}>
                      {inputs.side === 'BUY' ? 'Total Cash Out' : 'Net Proceeds'}
                    </div>
                    <div className={`text-xl font-bold ${inputs.side === 'BUY' ? 'text-emerald-700' : 'text-blue-700'}`}>
                      {formatCurrency(results.netAmount)}
                    </div>
                  </div>
                </div>

                {/* Effective Rate */}
                <div className="mb-6 flex justify-center">
                  <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-slate-100 text-slate-600 text-sm font-medium">
                    <span>Effective {inputs.side === 'BUY' ? 'Cost' : 'Price'}/Share:</span>
                    <span className="font-bold text-slate-800">{formatCurrency(results.effectivePrice)}</span>
                  </div>
                </div>

                {/* Friction Table */}
                <div className="overflow-x-auto">
                  <table className="w-full text-sm">
                    <thead className="bg-slate-50 text-slate-500 font-medium">
                      <tr>
                        <th className="text-left py-2 px-3 rounded-l-lg">Fee Component</th>
                        <th className="text-right py-2 px-3 rounded-r-lg">Amount (PHP)</th>
                      </tr>
                    </thead>
                    <tbody className="text-slate-700 divide-y divide-slate-100">
                      <tr>
                        <td className="py-2 px-3 flex flex-col">
                          <span>Broker Commission</span>
                          <span className="text-xs text-slate-400 font-normal">
                            {settings.commissionRate}% of gross (min {settings.minCommission})
                          </span>
                        </td>
                        <td className="text-right py-2 px-3 font-mono">{formatCurrency(results.commission)}</td>
                      </tr>
                      <tr>
                        <td className="py-2 px-3">VAT on Commission (12%)</td>
                        <td className="text-right py-2 px-3 font-mono">{formatCurrency(results.vatOnComm)}</td>
                      </tr>
                      <tr>
                        <td className="py-2 px-3">PSE Transaction Fee (0.005%)</td>
                        <td className="text-right py-2 px-3 font-mono">{formatCurrency(results.pseFee)}</td>
                      </tr>
                      {settings.applyVatOnPse && (
                        <tr>
                          <td className="py-2 px-3">VAT on PSE Fee (12%)</td>
                          <td className="text-right py-2 px-3 font-mono">{formatCurrency(results.vatOnPse)}</td>
                        </tr>
                      )}
                      <tr>
                        <td className="py-2 px-3">SCCP Fee (0.01%)</td>
                        <td className="text-right py-2 px-3 font-mono">{formatCurrency(results.sccpFee)}</td>
                      </tr>
                      {inputs.side === 'SELL' && (
                        <tr className="bg-rose-50/50">
                          <td className="py-2 px-3 font-medium text-rose-700">Stock Transaction Tax (0.1%)</td>
                          <td className="text-right py-2 px-3 font-mono text-rose-700 font-medium">{formatCurrency(results.salesTax)}</td>
                        </tr>
                      )}
                    </tbody>
                    <tfoot className="border-t-2 border-slate-100">
                      <tr>
                        <td className="py-3 px-3 font-bold text-slate-800">Total Friction Costs</td>
                        <td className="py-3 px-3 font-bold text-slate-800 text-right font-mono">{formatCurrency(results.totalFees)}</td>
                      </tr>
                    </tfoot>
                  </table>
                </div>
              </div>
            )}

            {/* Empty State Placeholder for Right Side */}
            {!results && (
              <div className="flex-1 bg-slate-50 rounded-xl border-2 border-dashed border-slate-200 flex flex-col items-center justify-center text-slate-400 min-h-[300px]">
                <Calculator className="w-12 h-12 mb-3 opacity-50" />
                <p className="font-medium">Enter trade details to calculate</p>
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
}
