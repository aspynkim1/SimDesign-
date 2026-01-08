import React, { useState, useEffect } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';
import { getFirestore, collection, addDoc, onSnapshot, query, serverTimestamp } from 'firebase/firestore';
import { 
  CheckCircle, FolderOpen, AlertTriangle, FileText, 
  Send, Download, Check, Lock, ChevronLeft, Database, Mail
} from 'lucide-react';

// Firebase Configuration
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'scenario-manager-v2';

export default function App() {
  const [view, setView] = useState('guide'); 
  const [user, setUser] = useState(null);
  const [inquiries, setInquiries] = useState([]);
  const [adminPassword, setAdminPassword] = useState('');
  const [isAdminAuthenticated, setIsAdminAuthenticated] = useState(false);
  
  // Form & Checklist State
  const [formData, setFormData] = useState({ institution: '', name: '', email: '', content: '' });
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [message, setMessage] = useState({ type: '', text: '' });
  const [checklist, setChecklist] = useState([
    { id: 1, text: "모든 의사 처방지/검사지가 .jpg 인가요?", checked: false },
    { id: 2, text: "모든 단서 음성이 .wav 형식인가요?", checked: false },
    { id: 3, text: "모든 AI 제작 영상이 .mp4 형식인가요?", checked: false },
    { id: 4, text: "파일명이 모두 영문으로 작성되었나요?", checked: false },
    { id: 5, text: "전체 흐름도(HWP/DOCX)가 포함되었나요?", checked: false },
  ]);

  // Auth & Initialization
  useEffect(() => {
    const initAuth = async () => {
      if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
        await signInWithCustomToken(auth, __initial_auth_token);
      } else {
        await signInAnonymously(auth);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  // Sync Inquiries for Admin
  useEffect(() => {
    if (!user || view !== 'admin') return;

    const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'inquiries'));
    const unsubscribe = onSnapshot(q, 
      (snapshot) => {
        const data = snapshot.docs.map(doc => ({
          id: doc.id,
          ...doc.data(),
          date: doc.data().createdAt?.toDate().toLocaleString() || '처리 중...'
        }));
        setInquiries(data);
      },
      (error) => console.error("Firestore Error:", error)
    );
    return () => unsubscribe();
  }, [user, view]);

  const handleInquirySubmit = async (e) => {
    e.preventDefault();
    if (!user) return;

    setIsSubmitting(true);
    try {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'inquiries'), {
        ...formData,
        createdAt: serverTimestamp()
      });
      setFormData({ institution: '', name: '', email: '', content: '' });
      setMessage({ type: 'success', text: '문의사항이 성공적으로 접수되었습니다!' });
      setTimeout(() => setMessage({ type: '', text: '' }), 5000);
    } catch (err) {
      setMessage({ type: 'error', text: '전송 중 오류가 발생했습니다. 다시 시도해 주세요.' });
    } finally {
      setIsSubmitting(false);
    }
  };

  const downloadExcel = () => {
    if (inquiries.length === 0) return;
    const headers = ["접수시간", "기관명", "성함", "이메일", "문의내용"];
    const rows = inquiries.map(iq => [
      `"${iq.date}"`,
      `"${iq.institution}"`,
      `"${iq.name}"`,
      `"${iq.email}"`,
      `"${iq.content.replace(/\n/g, ' ')}"`
    ]);
    const csvContent = "\uFEFF" + [headers, ...rows].map(e => e.join(",")).join("\n");
    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement("a");
    link.href = URL.createObjectURL(blob);
    link.download = `SimDesign_문의내역_${new Date().toISOString().split('T')[0]}.csv`;
    link.click();
  };

  const toggleCheck = (id) => {
    setChecklist(checklist.map(item => item.id === id ? { ...item, checked: !item.checked } : item));
  };

  const handleAdminLogin = (e) => {
    e.preventDefault();
    if (adminPassword === "sim1234") { 
      setIsAdminAuthenticated(true);
      setAdminPassword('');
    } else {
      setMessage({ type: 'error', text: '비밀번호가 일치하지 않습니다.' });
    }
  };

  // --- View Components ---

  const GuideView = () => (
    <div className="max-w-4xl mx-auto space-y-8 animate-in fade-in duration-500">
      <header className="bg-gradient-to-r from-blue-700 to-blue-500 rounded-2xl p-8 text-white shadow-lg">
        <h1 className="text-3xl font-bold flex items-center gap-3">
          <FolderOpen className="w-8 h-8" /> SimDesign 시나리오 준비 가이드
        </h1>
        <p className="mt-2 text-blue-100">원활한 시스템 등록을 위해 아래 준비 사항을 확인해 주세요.</p>
      </header>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
        <div className="md:col-span-2 space-y-6">
          <section className="bg-white rounded-xl p-6 shadow-sm border border-slate-200">
            <h2 className="text-xl font-semibold mb-4 flex items-center gap-2">
              <FileText className="text-blue-600" /> 표준 폴더 및 파일 구성
            </h2>
            <div className="bg-slate-900 text-slate-300 p-5 rounded-lg font-mono text-sm leading-relaxed border-2 border-slate-700">
              <div className="text-blue-400">📁 Scenario_Root_Folder</div>
              <div className="pl-4">├─ 📂 00_Overview (전체 흐름도.hwp)</div>
              <div className="pl-4">├─ 📂 01_Stage_Initial</div>
              <div className="pl-8 text-yellow-400">├─ 🖼️ Doctor_Order.jpg</div>
              <div className="pl-8 text-yellow-400">├─ 🖼️ Lab_Results.jpg</div>
              <div className="pl-8 text-pink-400">├─ 🎵 Cue_Audio.wav</div>
              <div className="pl-8 text-green-400">├─ 🎬 AI_Video.mp4</div>
              <div className="pl-4">└─ 📂 99_Media_Master</div>
            </div>
          </section>

          <section className="bg-white rounded-xl p-6 shadow-sm border border-slate-200">
            <h2 className="text-xl font-semibold mb-4 flex items-center gap-2">
              <CheckCircle className="text-green-600" /> 준비 상태 체크리스트
            </h2>
            <div className="grid gap-3">
              {checklist.map(item => (
                <button
                  key={item.id}
                  onClick={() => toggleCheck(item.id)}
                  className={`flex items-center justify-between p-4 rounded-xl border-2 transition-all ${
                    item.checked ? 'bg-green-50 border-green-500 text-green-800' : 'bg-white border-slate-100 hover:border-blue-200'
                  }`}
                >
                  <span className="font-medium text-left">{item.text}</span>
                  <div className={`w-6 h-6 rounded-full border-2 flex items-center justify-center shrink-0 ${
                    item.checked ? 'bg-green-500 border-green-500 text-white' : 'border-slate-300'
                  }`}>
                    {item.checked && <Check className="w-4 h-4" />}
                  </div>
                </button>
              ))}
            </div>
          </section>

          <div className="bg-amber-50 border-l-4 border-amber-500 p-5 rounded-r-xl flex gap-4">
            <AlertTriangle className="text-amber-500 shrink-0" />
            <div>
              <h4 className="font-bold text-amber-900">시스템 오류 방지 주의사항</h4>
              <p className="text-sm text-amber-800 mt-1">
                모든 <strong>미디어 파일(JPG, WAV, MP4)</strong>은 반드시 영문 파일명을 사용해야 합니다. 
                한글 파일명은 서버 로딩 시 오류의 원인이 됩니다.
              </p>
            </div>
          </div>
        </div>

        <div className="space-y-6">
          <section className="bg-white rounded-xl p-6 shadow-xl border border-blue-100 sticky top-8">
            <h2 className="text-xl font-semibold mb-4 flex items-center gap-2">
              <Send className="text-blue-600 w-5 h-5" /> 문의사항 접수
            </h2>
            <form onSubmit={handleInquirySubmit} className="space-y-4">
              <div>
                <label className="text-xs font-bold text-slate-500 uppercase">기관명</label>
                <input
                  required
                  className="w-full mt-1 p-2.5 border border-slate-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none text-sm"
                  value={formData.institution}
                  onChange={e => setFormData({...formData, institution: e.target.value})}
                />
              </div>
              <div>
                <label className="text-xs font-bold text-slate-500 uppercase">성함</label>
                <input
                  required
                  className="w-full mt-1 p-2.5 border border-slate-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none text-sm"
                  value={formData.name}
                  onChange={e => setFormData({...formData, name: e.target.value})}
                />
              </div>
              <div>
                <label className="text-xs font-bold text-slate-500 uppercase">답변 받으실 이메일</label>
                <input
                  type="email"
                  required
                  placeholder="example@email.com"
                  className="w-full mt-1 p-2.5 border border-slate-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none text-sm"
                  value={formData.email}
                  onChange={e => setFormData({...formData, email: e.target.value})}
                />
              </div>
              <div>
                <label className="text-xs font-bold text-slate-500 uppercase">문의내용</label>
                <textarea
                  required
                  rows="3"
                  className="w-full mt-1 p-2.5 border border-slate-200 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none text-sm"
                  value={formData.content}
                  onChange={e => setFormData({...formData, content: e.target.value})}
                />
              </div>
              
              <div className="p-3 bg-slate-50 rounded-lg border border-slate-100">
                <p className="text-[11px] text-slate-500 leading-relaxed italic">
                  * 추신: 본 문의에 대한 답변은 입력해주신 <strong>이메일</strong>을 통해서만 발송됩니다. 정확한 이메일 주소를 입력하셨는지 확인 부탁드립니다.
                </p>
              </div>

              <button
                type="submit"
                disabled={isSubmitting}
                className="w-full bg-blue-600 hover:bg-blue-700 text-white py-3 rounded-lg font-bold shadow-md transition-all active:scale-95 disabled:opacity-50"
              >
                {isSubmitting ? '전송 중...' : '문의하기'}
              </button>
              {message.text && (
                <div className={`p-3 rounded-lg text-sm text-center ${message.type === 'success' ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}`}>
                  {message.text}
                </div>
              )}
            </form>
          </section>
        </div>
      </div>

      <footer className="pt-12 pb-6 text-center border-t border-slate-200">
        <p className="text-slate-400 text-sm">© 2024 SimDesign Simulation. All rights reserved.</p>
        <button 
          onClick={() => setView('admin')}
          className="mt-4 text-slate-300 hover:text-slate-400 transition-colors"
        >
          <Lock className="w-4 h-4 mx-auto" />
        </button>
      </footer>
    </div>
  );

  const AdminView = () => (
    <div className="max-w-6xl mx-auto space-y-6 animate-in slide-in-from-bottom-4 duration-500">
      <div className="flex items-center justify-between">
        <button onClick={() => {setView('guide'); setIsAdminAuthenticated(false);}} className="flex items-center gap-2 text-slate-500 hover:text-blue-600 transition-colors">
          <ChevronLeft /> 가이드로 돌아가기
        </button>
        <h1 className="text-2xl font-bold flex items-center gap-2">
          <Database className="text-blue-600" /> 관리자 데이터 센터
        </h1>
      </div>

      {!isAdminAuthenticated ? (
        <div className="bg-white p-12 rounded-2xl shadow-xl border border-slate-200 text-center max-w-md mx-auto">
          <div className="w-16 h-16 bg-blue-50 rounded-full flex items-center justify-center mx-auto mb-6">
            <Lock className="text-blue-600" />
          </div>
          <h2 className="text-xl font-bold mb-2">관리자 인증</h2>
          <p className="text-slate-500 mb-6 text-sm">비밀번호를 입력하세요.</p>
          <form onSubmit={handleAdminLogin} className="space-y-4">
            <input
              type="password"
              placeholder="비밀번호"
              autoFocus
              className="w-full p-3 border border-slate-300 rounded-lg outline-none focus:ring-2 focus:ring-blue-500"
              value={adminPassword}
              onChange={e => setAdminPassword(e.target.value)}
            />
            <button className="w-full bg-slate-900 text-white py-3 rounded-lg font-bold">인증하기</button>
          </form>
          {message.text && <p className="mt-4 text-red-500 text-sm">{message.text}</p>}
        </div>
      ) : (
        <div className="space-y-4">
          <div className="flex justify-between items-end">
            <p className="text-slate-500 text-sm">현재 총 <span className="text-blue-600 font-bold">{inquiries.length}</span>건의 문의가 접수되었습니다.</p>
            <button 
              onClick={downloadExcel}
              className="flex items-center gap-2 bg-green-600 hover:bg-green-700 text-white px-6 py-2.5 rounded-lg font-bold shadow-lg transition-all"
            >
              <Download className="w-5 h-5" /> 엑셀(CSV) 다운로드
            </button>
          </div>

          <div className="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden overflow-x-auto">
            <table className="w-full text-left border-collapse min-w-[800px]">
              <thead>
                <tr className="bg-slate-50 border-b border-slate-200">
                  <th className="p-4 font-bold text-slate-600 text-xs uppercase">접수시간</th>
                  <th className="p-4 font-bold text-slate-600 text-xs uppercase">기관명</th>
                  <th className="p-4 font-bold text-slate-600 text-xs uppercase">성함</th>
                  <th className="p-4 font-bold text-slate-600 text-xs uppercase">이메일</th>
                  <th className="p-4 font-bold text-slate-600 text-xs uppercase">문의내용</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-100">
                {inquiries.map(iq => (
                  <tr key={iq.id} className="hover:bg-slate-50 transition-colors">
                    <td className="p-4 text-[11px] text-slate-400 whitespace-nowrap">{iq.date}</td>
                    <td className="p-4 font-semibold text-slate-700 text-sm">{iq.institution}</td>
                    <td className="p-4 text-slate-700 text-sm">{iq.name}</td>
                    <td className="p-4 text-blue-600 text-sm italic underline flex items-center gap-1">
                      <Mail className="w-3 h-3" /> {iq.email}
                    </td>
                    <td className="p-4 text-slate-600 text-sm leading-relaxed">{iq.content}</td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      )}
    </div>
  );

  return (
    <div className="min-h-screen bg-[#f8fafc] p-4 md:p-8">
      {view === 'guide' ? <GuideView /> : <AdminView />}
    </div>
  );
}
