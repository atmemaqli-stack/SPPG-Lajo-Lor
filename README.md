react
import React, { useState, useEffect } from 'react';
import { 
  LayoutDashboard, 
  School, 
  Baby, 
  ClipboardList, 
  Menu, 
  X, 
  CheckCircle2, 
  AlertCircle,
  Users,
  Activity
} from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, collection, addDoc, onSnapshot, serverTimestamp } from 'firebase/firestore';

// --- FIREBASE INITIALIZATION ---
// Menggunakan konfigurasi yang disediakan oleh environment
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
  apiKey: "demo-key",
  projectId: "demo-project",
};
const appId = typeof __app_id !== 'undefined' ? __app_id : 'sppg-lajo-lor-app';

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

export default function App() {
  const [user, setUser] = useState(null);
  const [activeTab, setActiveTab] = useState('dashboard');
  const [isSidebarOpen, setIsSidebarOpen] = useState(false);
  
  // Data State
  const [schoolData, setSchoolData] = useState([]);
  const [posyanduData, setPosyanduData] = useState([]);
  
  // Toast Notification State
  const [toast, setToast] = useState({ show: false, message: '', type: 'success' });

  // --- AUTHENTICATION & DATA FETCHING ---
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Auth error:", error);
      }
    };
    initAuth();

    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
    });
    return () => unsubscribe();
  }, []);

  useEffect(() => {
    if (!user) return;

    // Listener Data Sekolah
    const schoolRef = collection(db, 'artifacts', appId, 'public', 'data', 'schools');
    const unsubSchool = onSnapshot(schoolRef, (snapshot) => {
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      data.sort((a, b) => (b.createdAt?.toMillis() || 0) - (a.createdAt?.toMillis() || 0));
      setSchoolData(data);
    }, (error) => console.error("Error fetching school data:", error));

    // Listener Data Posyandu
    const posyanduRef = collection(db, 'artifacts', appId, 'public', 'data', 'posyandu');
    const unsubPosyandu = onSnapshot(posyanduRef, (snapshot) => {
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      data.sort((a, b) => (b.createdAt?.toMillis() || 0) - (a.createdAt?.toMillis() || 0));
      setPosyanduData(data);
    }, (error) => console.error("Error fetching posyandu data:", error));

    return () => {
      unsubSchool();
      unsubPosyandu();
    };
  }, [user]);

  // --- HELPER FUNCTIONS ---
  const showToast = (message, type = 'success') => {
    setToast({ show: true, message, type });
    setTimeout(() => setToast({ show: false, message: '', type: 'success' }), 3000);
  };

  const closeSidebar = () => {
    if (window.innerWidth < 1024) {
      setIsSidebarOpen(false);
    }
  };

  // --- COMPONENTS ---
  const SidebarItem = ({ icon: Icon, label, id }) => (
    <button
      onClick={() => { setActiveTab(id); closeSidebar(); }}
      className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl transition-all ${
        activeTab === id 
          ? 'bg-emerald-50 text-emerald-700 font-semibold' 
          : 'text-slate-600 hover:bg-slate-50 hover:text-emerald-600'
      }`}
    >
      <Icon className={`w-5 h-5 ${activeTab === id ? 'text-emerald-600' : 'text-slate-400'}`} />
      <span>{label}</span>
    </button>
  );

  return (
    <div className="min-h-screen bg-slate-50 flex font-sans text-slate-800">
      
      {/* TOAST NOTIFICATION */}
      <div className={`fixed top-4 right-4 z-[60] flex items-center p-4 rounded-xl shadow-lg border text-white transition-all duration-300 transform ${toast.show ? 'translate-y-0 opacity-100' : '-translate-y-full opacity-0'} ${toast.type === 'success' ? 'bg-emerald-600 border-emerald-500' : 'bg-rose-600 border-rose-500'}`}>
        {toast.type === 'success' ? <CheckCircle2 className="w-5 h-5 mr-2" /> : <AlertCircle className="w-5 h-5 mr-2" />}
        <span className="font-medium text-sm">{toast.message}</span>
      </div>

      {/* MOBILE OVERLAY */}
      {isSidebarOpen && (
        <div 
          className="fixed inset-0 bg-slate-800/50 z-40 lg:hidden backdrop-blur-sm"
          onClick={() => setIsSidebarOpen(false)}
        />
      )}

      {/* SIDEBAR */}
      <aside className={`fixed lg:static inset-y-0 left-0 z-50 w-72 bg-white border-r border-slate-200 transform transition-transform duration-300 flex flex-col ${isSidebarOpen ? 'translate-x-0' : '-translate-x-full lg:translate-x-0'}`}>
        <div className="p-6 border-b border-slate-100 flex items-center justify-between">
          <div>
            <h1 className="text-xl font-bold text-slate-800 tracking-tight">SPPG Lajo Lor</h1>
            <p className="text-xs text-slate-500 mt-1">Wilayah Singgahan</p>
          </div>
          <button onClick={() => setIsSidebarOpen(false)} className="lg:hidden p-2 text-slate-400 hover:text-slate-600 bg-slate-50 rounded-lg">
            <X className="w-5 h-5" />
          </button>
        </div>
        
        <div className="flex-1 overflow-y-auto py-6 px-4 space-y-2">
          <p className="px-4 text-xs font-semibold text-slate-400 uppercase tracking-wider mb-4">Menu Utama</p>
          <SidebarItem icon={LayoutDashboard} label="Dashboard" id="dashboard" />
          <SidebarItem icon={School} label="Input Data Sekolah" id="form-school" />
          <SidebarItem icon={Baby} label="Input Data Posyandu" id="form-posyandu" />
          
          <div className="pt-6 pb-2">
            <p className="px-4 text-xs font-semibold text-slate-400 uppercase tracking-wider mb-4">Rekapitulasi</p>
          </div>
          <SidebarItem icon={ClipboardList} label="Data Sekolah" id="data-school" />
          <SidebarItem icon={ClipboardList} label="Data Posyandu" id="data-posyandu" />
        </div>
        
        <div className="p-4 border-t border-slate-100">
          <div className="bg-emerald-50 rounded-xl p-4 flex items-center space-x-3">
            <div className="w-2 h-2 bg-emerald-500 rounded-full animate-pulse" />
            <span className="text-xs font-medium text-emerald-700">Sistem Online & Real-time</span>
          </div>
        </div>
      </aside>

      {/* MAIN CONTENT */}
      <main className="flex-1 flex flex-col min-w-0 overflow-hidden">
        {/* HEADER MOBILE */}
        <header className="bg-white border-b border-slate-200 p-4 flex items-center justify-between lg:hidden sticky top-0 z-30">
          <div className="flex items-center space-x-3">
            <button onClick={() => setIsSidebarOpen(true)} className="p-2 text-slate-600 hover:bg-slate-100 rounded-lg">
              <Menu className="w-6 h-6" />
            </button>
            <h1 className="text-lg font-bold text-slate-800">SPPG Lajo Lor</h1>
          </div>
        </header>

        {/* CONTENT AREA */}
        <div className="flex-1 overflow-y-auto p-4 lg:p-8">
          <div className="max-w-6xl mx-auto">
            
            {/* --- VIEW: DASHBOARD --- */}
            {activeTab === 'dashboard' && (
              <div className="space-y-6">
                <div>
                  <h2 className="text-2xl font-bold text-slate-800">Dashboard Terpadu</h2>
                  <p className="text-slate-500 mt-1">Ringkasan data wilayah Singgahan secara real-time.</p>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-4 gap-6">
                  <StatCard 
                    title="Total Siswa Terdata" 
                    count={schoolData.filter(d => d.kategori === 'Siswa').length} 
                    icon={Users} 
                    color="blue"
                  />
                  <StatCard 
                    title="Guru & PIC" 
                    count={schoolData.filter(d => d.kategori !== 'Siswa').length} 
                    icon={School} 
                    color="indigo"
                  />
                  <StatCard 
                    title="Balita Terdata" 
                    count={posyanduData.filter(d => d.kategori === 'Balita').length} 
                    icon={Baby} 
                    color="emerald"
                  />
                  <StatCard 
                    title="Ibu Hamil & Menyusui" 
                    count={posyanduData.filter(d => d.kategori !== 'Balita').length} 
                    icon={Activity} 
                    color="rose"
                  />
                </div>

                <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mt-8">
                  <div className="bg-white rounded-2xl p-6 shadow-sm border border-slate-200">
                    <h3 className="text-lg font-bold text-slate-800 mb-4">Aktivitas Terbaru Sekolah</h3>
                    <div className="space-y-4">
                      {schoolData.slice(0, 5).map((item, i) => (
                        <div key={i} className="flex items-center space-x-4 border-b border-slate-50 pb-4 last:pb-0 last:border-0">
                          <div className="w-10 h-10 rounded-full bg-blue-50 flex items-center justify-center text-blue-600 font-bold">
                            {item.nama.charAt(0)}
                          </div>
                          <div>
                            <p className="text-sm font-semibold text-slate-800">{item.nama}</p>
                            <p className="text-xs text-slate-500">{item.kategori} - {item.instansi}</p>
                          </div>
                        </div>
                      ))}
                      {schoolData.length === 0 && <p className="text-sm text-slate-500">Belum ada data.</p>}
                    </div>
                  </div>

                  <div className="bg-white rounded-2xl p-6 shadow-sm border border-slate-200">
                    <h3 className="text-lg font-bold text-slate-800 mb-4">Aktivitas Terbaru Posyandu</h3>
                    <div className="space-y-4">
                      {posyanduData.slice(0, 5).map((item, i) => (
                        <div key={i} className="flex items-center space-x-4 border-b border-slate-50 pb-4 last:pb-0 last:border-0">
                          <div className="w-10 h-10 rounded-full bg-emerald-50 flex items-center justify-center text-emerald-600 font-bold">
                            {item.nama.charAt(0)}
                          </div>
                          <div>
                            <p className="text-sm font-semibold text-slate-800">{item.nama}</p>
                            <p className="text-xs text-slate-500">{item.kategori} - {item.namaPosyandu}</p>
                          </div>
                        </div>
                      ))}
                      {posyanduData.length === 0 && <p className="text-sm text-slate-500">Belum ada data.</p>}
                    </div>
                  </div>
                </div>
              </div>
            )}

            {/* --- VIEW: FORM SCHOOL --- */}
            {activeTab === 'form-school' && (
              <SchoolForm onSubmitSuccess={() => showToast("Data Sekolah berhasil disimpan!")} />
            )}

            {/* --- VIEW: FORM POSYANDU --- */}
            {activeTab === 'form-posyandu' && (
              <PosyanduForm onSubmitSuccess={() => showToast("Data Posyandu berhasil disimpan!")} />
            )}

            {/* --- VIEW: DATA SCHOOL --- */}
            {activeTab === 'data-school' && (
              <DataTableView 
                title="Rekapitulasi Data Sekolah" 
                data={schoolData} 
                columns={['Nama Lengkap', 'Kategori', 'Asal Instansi/Sekolah', 'No. HP', 'Alamat']}
                keys={['nama', 'kategori', 'instansi', 'nohp', 'alamat']}
              />
            )}

            {/* --- VIEW: DATA POSYANDU --- */}
            {activeTab === 'data-posyandu' && (
              <DataTableView 
                title="Rekapitulasi Data Posyandu" 
                data={posyanduData} 
                columns={['Nama Lengkap', 'Kategori', 'Nama Posyandu', 'Usia/Usia Kandungan', 'Alamat']}
                keys={['nama', 'kategori', 'namaPosyandu', 'usia', 'alamat']}
              />
            )}

          </div>
        </div>
      </main>
    </div>
  );
}

// --- SUB-COMPONENTS ---

function StatCard({ title, count, icon: Icon, color }) {
  const colorMap = {
    blue: 'bg-blue-50 text-blue-600',
    indigo: 'bg-indigo-50 text-indigo-600',
    emerald: 'bg-emerald-50 text-emerald-600',
    rose: 'bg-rose-50 text-rose-600'
  };

  return (
    <div className="bg-white rounded-2xl p-6 shadow-sm border border-slate-200 flex items-center space-x-4">
      <div className={`p-4 rounded-xl ${colorMap[color]}`}>
        <Icon className="w-6 h-6" />
      </div>
      <div>
        <p className="text-sm font-medium text-slate-500">{title}</p>
        <p className="text-2xl font-bold text-slate-800">{count}</p>
      </div>
    </div>
  );
}

function SchoolForm({ onSubmitSuccess }) {
  const [loading, setLoading] = useState(false);
  const [formData, setFormData] = useState({
    nama: '', kategori: 'Siswa', instansi: '', nohp: '', alamat: ''
  });

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!formData.nama || !formData.instansi) return;
    
    setLoading(true);
    try {
      await addDoc(collection(getFirestore(), 'artifacts', appId, 'public', 'data', 'schools'), {
        ...formData,
        createdAt: serverTimestamp()
      });
      onSubmitSuccess();
      setFormData({ nama: '', kategori: 'Siswa', instansi: '', nohp: '', alamat: '' });
    } catch (error) {
      console.error(error);
      alert("Gagal menyimpan data.");
    }
    setLoading(false);
  };

  return (
    <div className="max-w-2xl mx-auto bg-white rounded-2xl shadow-sm border border-slate-200 p-6 lg:p-8">
      <div className="flex items-center space-x-3 mb-6 border-b border-slate-100 pb-4">
        <div className="p-2 bg-blue-50 rounded-lg text-blue-600"><School className="w-6 h-6" /></div>
        <div>
          <h2 className="text-xl font-bold text-slate-800">Formulir Data Sekolah</h2>
          <p className="text-sm text-slate-500">Input data Siswa, Guru, atau PIC Sekolah wilayah Singgahan.</p>
        </div>
      </div>

      <form onSubmit={handleSubmit} className="space-y-5">
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-1">Nama Lengkap</label>
          <input required type="text" value={formData.nama} onChange={e => setFormData({...formData, nama: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition-all" placeholder="Masukkan nama lengkap" />
        </div>
        
        <div className="grid grid-cols-1 md:grid-cols-2 gap-5">
          <div>
            <label className="block text-sm font-medium text-slate-700 mb-1">Kategori</label>
            <select value={formData.kategori} onChange={e => setFormData({...formData, kategori: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition-all bg-white">
              <option>Siswa</option>
              <option>Guru</option>
              <option>PIC Sekolah</option>
            </select>
          </div>
          <div>
            <label className="block text-sm font-medium text-slate-700 mb-1">Asal Sekolah/Instansi</label>
            <input required type="text" value={formData.instansi} onChange={e => setFormData({...formData, instansi: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 outline-none transition-all" placeholder="Contoh: SDN 1 Singgahan" />
          </div>
        </div>

        <div>
          <label className="block text-sm font-medium text-slate-700 mb-1">No. WhatsApp / HP</label>
          <input type="tel" value={formData.nohp} onChange={e => setFormData({...formData, nohp: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 outline-none transition-all" placeholder="08xxxxxxxxxx (Opsional)" />
        </div>

        <div>
          <label className="block text-sm font-medium text-slate-700 mb-1">Alamat Lengkap</label>
          <textarea required rows="3" value={formData.alamat} onChange={e => setFormData({...formData, alamat: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 outline-none transition-all resize-none" placeholder="Masukkan alamat domisili..."></textarea>
        </div>

        <button disabled={loading} type="submit" className="w-full py-3 px-4 bg-emerald-600 hover:bg-emerald-700 text-white font-semibold rounded-xl shadow-sm transition-all focus:ring-2 focus:ring-offset-2 focus:ring-emerald-500 disabled:opacity-70">
          {loading ? 'Menyimpan Data...' : 'Simpan Data'}
        </button>
      </form>
    </div>
  );
}

function PosyanduForm({ onSubmitSuccess }) {
  const [loading, setLoading] = useState(false);
  const [formData, setFormData] = useState({
    nama: '', kategori: 'Balita', namaPosyandu: '', usia: '', alamat: ''
  });

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!formData.nama || !formData.namaPosyandu) return;
    
    setLoading(true);
    try {
      await addDoc(collection(getFirestore(), 'artifacts', appId, 'public', 'data', 'posyandu'), {
        ...formData,
        createdAt: serverTimestamp()
      });
      onSubmitSuccess();
      setFormData({ nama: '', kategori: 'Balita', namaPosyandu: '', usia: '', alamat: '' });
    } catch (error) {
      console.error(error);
      alert("Gagal menyimpan data.");
    }
    setLoading(false);
  };

  return (
    <div className="max-w-2xl mx-auto bg-white rounded-2xl shadow-sm border border-slate-200 p-6 lg:p-8">
      <div className="flex items-center space-x-3 mb-6 border-b border-slate-100 pb-4">
        <div className="p-2 bg-emerald-50 rounded-lg text-emerald-600"><Baby className="w-6 h-6" /></div>
        <div>
          <h2 className="text-xl font-bold text-slate-800">Formulir Data Posyandu</h2>
          <p className="text-sm text-slate-500">Input data Balita, Ibu Hamil, dan Menyusui wilayah Singgahan.</p>
        </div>
      </div>

      <form onSubmit={handleSubmit} className="space-y-5">
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-1">Nama Lengkap</label>
          <input required type="text" value={formData.nama} onChange={e => setFormData({...formData, nama: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 outline-none transition-all" placeholder="Nama Balita / Ibu" />
        </div>
        
        <div className="grid grid-cols-1 md:grid-cols-2 gap-5">
          <div>
            <label className="block text-sm font-medium text-slate-700 mb-1">Kategori Kunjungan</label>
            <select value={formData.kategori} onChange={e => setFormData({...formData, kategori: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 outline-none transition-all bg-white">
              <option>Balita</option>
              <option>Ibu Hamil</option>
              <option>Ibu Menyusui</option>
            </select>
          </div>
          <div>
            <label className="block text-sm font-medium text-slate-700 mb-1">Nama Posyandu (Desa/Dusun)</label>
            <input required type="text" value={formData.namaPosyandu} onChange={e => setFormData({...formData, namaPosyandu: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 outline-none transition-all" placeholder="Contoh: Posyandu Melati / Lajo Lor" />
          </div>
        </div>

        <div>
          <label className="block text-sm font-medium text-slate-700 mb-1">
            {formData.kategori === 'Balita' ? 'Usia Balita (Bulan/Tahun)' : 'Usia Kandungan / Bayi'}
          </label>
          <input required type="text" value={formData.usia} onChange={e => setFormData({...formData, usia: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 outline-none transition-all" placeholder="Contoh: 18 Bulan" />
        </div>

        <div>
          <label className="block text-sm font-medium text-slate-700 mb-1">Alamat Lengkap (RT/RW)</label>
          <textarea required rows="3" value={formData.alamat} onChange={e => setFormData({...formData, alamat: e.target.value})} className="w-full px-4 py-2.5 rounded-xl border border-slate-300 focus:ring-2 focus:ring-emerald-500 outline-none transition-all resize-none" placeholder="Masukkan alamat lengkap..."></textarea>
        </div>

        <button disabled={loading} type="submit" className="w-full py-3 px-4 bg-emerald-600 hover:bg-emerald-700 text-white font-semibold rounded-xl shadow-sm transition-all focus:ring-2 focus:ring-offset-2 focus:ring-emerald-500 disabled:opacity-70">
          {loading ? 'Menyimpan Data...' : 'Simpan Data'}
        </button>
      </form>
    </div>
  );
}

function DataTableView({ title, data, columns, keys }) {
  return (
    <div className="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden flex flex-col h-full">
      <div className="p-6 border-b border-slate-100 flex items-center justify-between bg-white">
        <div>
          <h2 className="text-xl font-bold text-slate-800">{title}</h2>
          <p className="text-sm text-slate-500 mt-1">Total {data.length} data tersimpan.</p>
        </div>
      </div>
      
      <div className="overflow-x-auto">
        <table className="w-full text-left border-collapse">
          <thead>
            <tr className="bg-slate-50 border-b border-slate-200">
              <th className="py-4 px-6 font-semibold text-slate-600 text-sm whitespace-nowrap">No</th>
              {columns.map((col, i) => (
                <th key={i} className="py-4 px-6 font-semibold text-slate-600 text-sm whitespace-nowrap">{col}</th>
              ))}
            </tr>
          </thead>
          <tbody className="divide-y divide-slate-100">
            {data.length > 0 ? (
              data.map((item, index) => (
                <tr key={item.id} className="hover:bg-slate-50/50 transition-colors">
                  <td className="py-4 px-6 text-sm text-slate-600">{index + 1}</td>
                  {keys.map((key, i) => (
                    <td key={i} className="py-4 px-6 text-sm text-slate-800 font-medium">
                      {item[key] || '-'}
                    </td>
                  ))}
                </tr>
              ))
            ) : (
              <tr>
                <td colSpan={columns.length + 1} className="py-12 text-center text-slate-500">
                  Belum ada data yang dimasukkan.
                </td>
              </tr>
            )}
          </tbody>
        </table>
      </div>
    </div>
  );
}
