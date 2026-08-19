import React, { useState, useEffect } from 'react';
import { 
  Plus, Calendar, Smile, Image as ImageIcon, Trash2, Edit, X, 
  Search, BookOpen, Heart, Coffee, Frown, Angry, Sparkles, Filter 
} from 'lucide-react';

// 기분 옵션 데이터
const MOOD_OPTIONS = [
  { emoji: '😊', label: '기쁨', color: 'bg-yellow-100 text-yellow-700 border-yellow-300' },
  { emoji: '🎉', label: '축하', color: 'bg-purple-100 text-purple-700 border-purple-300' },
  { emoji: '☕', label: '여유', color: 'bg-amber-100 text-amber-700 border-amber-300' },
  { emoji: '🌿', label: '힐링', color: 'bg-emerald-100 text-emerald-700 border-emerald-300' },
  { emoji: '😭', label: '슬픔', color: 'bg-blue-100 text-blue-700 border-blue-300' },
  { emoji: '😡', label: '화남', color: 'bg-rose-100 text-rose-700 border-rose-300' },
];

export default function DailyLogApp() {
  // --- 상태 관리 ---
  const [logs, setLogs] = useState([]);
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedMoodFilter, setSelectedMoodFilter] = useState('ALL');
  
  // 모달 상태
  const [isFormOpen, setIsFormOpen] = useState(false);
  const [editingLog, setEditingLog] = useState(null); // 수정할 일기 (null이면 작성)
  const [selectedLog, setSelectedLog] = useState(null); // 상세 보기 모달 일기

  // 폼 입력 상태
  const [formData, setFormData] = useState({
    date: new Date().toISOString().substring(0, 10),
    mood: '😊',
    title: '',
    content: '',
    imageBase64: ''
  });

  // --- 1. 초기 LocalStorage 불러오기 ---
  useEffect(() => {
    const savedLogs = localStorage.getItem('daily_logs');
    if (savedLogs) {
      try {
        setLogs(JSON.parse(savedLogs));
      } catch (e) {
        console.error('Failed to parse logs from LocalStorage', e);
      }
    } else {
      // 샘플 초기 데이터 생성
      const sampleLogs = [
        {
          id: '1',
          date: new Date().toISOString().substring(0, 10),
          mood: '☕',
          title: '따뜻한 라떼 한 잔과 카페 산책',
          content: '오랜만에 가고 싶었던 동네 카페에 들렀다. 창가 자리에 앉아 고요하게 책을 읽고 일기를 쓰니 마음이 한결 가벼워지는 느낌이었다.',
          imageBase64: 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?auto=format&fit=crop&w=600&q=80',
          createdAt: new Date().toISOString()
        }
      ];
      setLogs(sampleLogs);
      localStorage.setItem('daily_logs', JSON.stringify(sampleLogs));
    }
  }, []);

  // --- 2. LocalStorage 동기화 ---
  const saveLogsToStorage = (updatedLogs) => {
    setLogs(updatedLogs);
    localStorage.setItem('daily_logs', JSON.stringify(updatedLogs));
  };

  // --- 3. 이미지 업로드 처리 (FileReader) ---
  const handleImageChange = (e) => {
    const file = e.target.files[0];
    if (file) {
      if (file.size > 2 * 1024 * 1024) {
        alert('이미지 크기는 2MB 이하만 업로드 가능합니다.');
        return;
      }
      const reader = new FileReader();
      reader.onloadend = () => {
        setFormData(prev => ({ ...prev, imageBase64: reader.result }));
      };
      reader.readAsDataURL(file);
    }
  };

  // --- 4. 폼 제출 (생성 및 수정) ---
  const handleSubmit = (e) => {
    e.preventDefault();
    if (!formData.title.trim() || !formData.content.trim()) {
      alert('제목과 본문 내용을 모두 입력해주세요.');
      return;
    }

    if (editingLog) {
      // 수정
      const updated = logs.map(item => 
        item.id === editingLog.id ? { ...item, ...formData } : item
      );
      saveLogsToStorage(updated);
      if (selectedLog?.id === editingLog.id) {
        setSelectedLog({ ...editingLog, ...formData });
      }
    } else {
      // 새로 작성
      const newEntry = {
        id: Date.now().toString(),
        ...formData,
        createdAt: new Date().toISOString()
      };
      saveLogsToStorage([newEntry, ...logs]);
    }

    closeFormModal();
  };

  // --- 5. 삭제 처리 ---
  const handleDelete = (id) => {
    if (window.confirm('정말로 이 일기를 삭제하시겠습니까?')) {
      const filtered = logs.filter(item => item.id !== id);
      saveLogsToStorage(filtered);
      if (selectedLog?.id === id) setSelectedLog(null);
    }
  };

  // --- 6. 모달 제어 함수 ---
  const openNewForm = () => {
    setEditingLog(null);
    setFormData({
      date: new Date().toISOString().substring(0, 10),
      mood: '😊',
      title: '',
      content: '',
      imageBase64: ''
    });
    setIsFormOpen(true);
  };

  const openEditForm = (log) => {
    setEditingLog(log);
    setFormData({
      date: log.date,
      mood: log.mood,
      title: log.title,
      content: log.content,
      imageBase64: log.imageBase64 || ''
    });
    setIsFormOpen(true);
  };

  const closeFormModal = () => {
    setIsFormOpen(false);
    setEditingLog(null);
  };

  // --- 7. 필터링 및 정렬 ---
  const filteredLogs = logs
    .filter(log => {
      const matchesSearch = log.title.toLowerCase().includes(searchQuery.toLowerCase()) || 
                            log.content.toLowerCase().includes(searchQuery.toLowerCase());
      const matchesMood = selectedMoodFilter === 'ALL' || log.mood === selectedMoodFilter;
      return matchesSearch && matchesMood;
    })
    .sort((a, b) => new Date(b.date) - new Date(a.date));

  return (
    <div className="min-h-screen bg-amber-50/40 text-stone-800 font-sans pb-12">
      {/* GNB / 상단 헤더 */}
      <header className="sticky top-0 z-20 bg-white/80 backdrop-blur-md border-b border-stone-200">
        <div className="max-w-4xl mx-auto px-4 py-4 flex items-center justify-between">
          <div className="flex items-center space-x-3">
            <div className="p-2 bg-amber-100 rounded-2xl text-amber-800">
              <BookOpen className="w-6 h-6" />
            </div>
            <div>
              <h1 className="text-xl font-bold tracking-tight text-stone-800">Daily Log</h1>
              <p className="text-xs text-stone-500">나만의 소소한 일상 기록</p>
            </div>
          </div>
          
          <button
            onClick={openNewForm}
            className="flex items-center space-x-1.5 px-4 py-2.5 bg-stone-800 hover:bg-stone-700 text-white text-sm font-medium rounded-full shadow-sm hover:shadow transition-all"
          >
            <Plus className="w-4 h-4" />
            <span>오늘 일기 쓰기</span>
          </button>
        </div>
      </header>

      {/* 메인 콘텐츠 영역 */}
      <main className="max-w-4xl mx-auto px-4 pt-6 space-y-6">
        
        {/* 필터 및 검색 바 */}
        <div className="bg-white p-4 rounded-2xl border border-stone-200 shadow-sm flex flex-col md:flex-row gap-3 items-center justify-between">
          {/* 검색 창 */}
          <div className="relative w-full md:w-72">
            <Search className="w-4 h-4 absolute left-3 top-1/2 -translate-y-1/2 text-stone-400" />
            <input
              type="text"
              placeholder="일기 내용 검색..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              className="w-full pl-9 pr-4 py-2 text-sm bg-stone-50 border border-stone-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-amber-500/50"
            />
          </div>

          {/* 기분별 필터 이모지 */}
          <div className="flex items-center space-x-1.5 overflow-x-auto w-full md:w-auto pb-1 md:pb-0">
            <button
              onClick={() => setSelectedMoodFilter('ALL')}
              className={`px-3 py-1.5 text-xs font-medium rounded-xl transition-all ${
                selectedMoodFilter === 'ALL'
                  ? 'bg-stone-800 text-white'
                  : 'bg-stone-100 text-stone-600 hover:bg-stone-200'
              }`}
            >
              전체 ({logs.length})
            </button>
            {MOOD_OPTIONS.map((m) => (
              <button
                key={m.emoji}
                onClick={() => setSelectedMoodFilter(m.emoji)}
                className={`px-2.5 py-1.5 text-xs font-medium rounded-xl border flex items-center space-x-1 transition-all ${
                  selectedMoodFilter === m.emoji
                    ? 'border-amber-500 bg-amber-50 font-bold'
                    : 'border-stone-200 bg-white hover:bg-stone-50 text-stone-600'
                }`}
              >
                <span>{m.emoji}</span>
              </button>
            ))}
          </div>
        </div>

        {/* 타임라인 목록 카드 */}
        {filteredLogs.length === 0 ? (
          <div className="text-center py-20 bg-white rounded-3xl border border-stone-200 shadow-sm space-y-3">
            <Sparkles className="w-10 h-10 mx-auto text-amber-400" />
            <p className="text-stone-500 font-medium">작성된 일기가 없습니다.</p>
            <p className="text-xs text-stone-400">오늘의 소중한 순간을 첫 일기로 기록해보세요!</p>
          </div>
        ) : (
          <div className="space-y-4">
            {filteredLogs.map((log) => (
              <article
                key={log.id}
                onClick={() => setSelectedLog(log)}
                className="group bg-white rounded-2xl p-5 border border-stone-200/80 shadow-sm hover:shadow-md transition-all cursor-pointer flex flex-col md:flex-row gap-5 items-start"
              >
                {/* 썸네일 이미지 (있을 때만) */}
                {log.imageBase64 && (
                  <div className="w-full md:w-36 h-36 md:h-28 rounded-xl overflow-hidden bg-stone-100 flex-shrink-0">
                    <img
                      src={log.imageBase64}
                      alt={log.title}
                      className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
                    />
                  </div>
                )}

                {/* 콘텐츠 요약 */}
                <div className="flex-1 min-w-0 space-y-2 w-full">
                  <div className="flex items-center justify-between">
                    <div className="flex items-center space-x-2">
                      <span className="text-xl" title="기분">{log.mood}</span>
                      <span className="text-xs font-semibold px-2.5 py-1 bg-stone-100 text-stone-600 rounded-lg flex items-center space-x-1">
                        <Calendar className="w-3 h-3 text-stone-400" />
                        <span>{log.date}</span>
                      </span>
                    </div>

                    {/* 카드 내 빠른 액션 */}
                    <div className="flex items-center space-x-1 opacity-80 md:opacity-0 group-hover:opacity-100 transition-opacity" onClick={(e) => e.stopPropagation()}>
                      <button
                        onClick={() => openEditForm(log)}
                        className="p-1.5 text-stone-400 hover:text-stone-700 hover:bg-stone-100 rounded-lg"
                        title="수정"
                      >
                        <Edit className="w-4 h-4" />
                      </button>
                      <button
                        onClick={() => handleDelete(log.id)}
                        className="p-1.5 text-stone-400 hover:text-rose-600 hover:bg-rose-50 rounded-lg"
                        title="삭제"
                      >
                        <Trash2 className="w-4 h-4" />
                      </button>
                    </div>
                  </div>

                  <h2 className="text-lg font-bold text-stone-800 truncate group-hover:text-amber-700 transition-colors">
                    {log.title}
                  </h2>

                  <p className="text-stone-600 text-sm line-clamp-2 leading-relaxed">
                    {log.content}
                  </p>
                </div>
              </article>
            ))}
          </div>
        )}
      </main>

      {/* --- MODAL 1: 작성 및 수정 모달 --- */}
      {isFormOpen && (
        <div className="fixed inset-0 z-50 bg-stone-900/40 backdrop-blur-sm flex items-center justify-center p-4">
          <div className="bg-white rounded-3xl max-w-lg w-full max-h-[90vh] overflow-y-auto p-6 shadow-2xl space-y-5 animate-in fade-in zoom-in-95 duration-200">
            <div className="flex items-center justify-between border-b border-stone-100 pb-3">
              <h3 className="text-lg font-bold text-stone-800">
                {editingLog ? '일기 수정하기' : '오늘 하루 기록하기'}
              </h3>
              <button onClick={closeFormModal} className="p-1 text-stone-400 hover:text-stone-600 rounded-full">
                <X className="w-5 h-5" />
              </button>
            </div>

            <form onSubmit={handleSubmit} className="space-y-4">
              {/* 날짜 선택 & 기분 선택 */}
              <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                <div>
                  <label className="block text-xs font-semibold text-stone-600 mb-1.5">날짜</label>
                  <input
                    type="date"
                    value={formData.date}
                    onChange={(e) => setFormData({ ...formData, date: e.target.value })}
                    className="w-full px-3 py-2 text-sm border border-stone-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-amber-500/50"
                  />
                </div>

                <div>
                  <label className="block text-xs font-semibold text-stone-600 mb-1.5">오늘의 기분</label>
                  <div className="flex space-x-1.5">
                    {MOOD_OPTIONS.map((m) => (
                      <button
                        key={m.emoji}
                        type="button"
                        onClick={() => setFormData({ ...formData, mood: m.emoji })}
                        className={`p-2 text-lg rounded-xl transition-all border ${
                          formData.mood === m.emoji
                            ? 'bg-amber-100 border-amber-400 scale-110 shadow-sm'
                            : 'border-stone-100 hover:bg-stone-50 opacity-60'
                        }`}
                      >
                        {m.emoji}
                      </button>
                    ))}
                  </div>
                </div>
              </div>

              {/* 제목 */}
              <div>
                <label className="block text-xs font-semibold text-stone-600 mb-1.5">제목</label>
                <input
                  type="text"
                  placeholder="오늘 하루를 한 문장으로 표현해보세요"
                  value={formData.title}
                  onChange={(e) => setFormData({ ...formData, title: e.target.value })}
                  className="w-full px-3.5 py-2.5 text-sm border border-stone-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-amber-500/50"
                />
              </div>

              {/* 본문 */}
              <div>
                <label className="block text-xs font-semibold text-stone-600 mb-1.5">일기 내용</label>
                <textarea
                  rows={5}
                  placeholder="어떤 일이 있으셨나요? 솔직한 감정을 기록해보세요..."
                  value={formData.content}
                  onChange={(e) => setFormData({ ...formData, content: e.target.value })}
                  className="w-full px-3.5 py-2.5 text-sm border border-stone-200 rounded-xl focus:outline-none focus:ring-2 focus:ring-amber-500/50 resize-none leading-relaxed"
                />
              </div>

              {/* 이미지 첨부 */}
              <div>
                <label className="block text-xs font-semibold text-stone-600 mb-1.5">사진 첨부 (선택)</label>
                <div className="flex items-center space-x-3">
                  <label className="flex items-center space-x-2 px-3.5 py-2 border border-stone-200 rounded-xl text-xs font-medium text-stone-600 hover:bg-stone-50 cursor-pointer transition-colors">
                    <ImageIcon className="w-4 h-4 text-stone-500" />
                    <span>사진 선택</span>
                    <input type="file" accept="image/*" onChange={handleImageChange} className="hidden" />
                  </label>

                  {formData.imageBase64 && (
                    <button
                      type="button"
                      onClick={() => setFormData({ ...formData, imageBase64: '' })}
                      className="text-xs text-rose-500 hover:underline"
                    >
                      사진 삭제
                    </button>
                  )}
                </div>

                {/* 이미지 미리보기 */}
                {formData.imageBase64 && (
                  <div className="mt-3 relative w-full h-40 rounded-xl overflow-hidden border border-stone-200 bg-stone-50">
                    <img src={formData.imageBase64} alt="미리보기" className="w-full h-full object-cover" />
                  </div>
                )}
              </div>

              {/* 하단 버튼 */}
              <div className="pt-3 flex justify-end space-x-2 border-t border-stone-100">
                <button
                  type="button"
                  onClick={closeFormModal}
                  className="px-4 py-2 text-xs font-medium text-stone-600 hover:bg-stone-100 rounded-xl"
                >
                  취소
                </button>
                <button
                  type="submit"
                  className="px-5 py-2 text-xs font-semibold bg-stone-800 text-white hover:bg-stone-700 rounded-xl shadow-sm"
                >
                  {editingLog ? '수정 완료' : '저장하기'}
                </button>
              </div>
            </form>
          </div>
        </div>
      )}

      {/* --- MODAL 2: 일기 상세 보기 모달 --- */}
      {selectedLog && (
        <div className="fixed inset-0 z-50 bg-stone-900/40 backdrop-blur-sm flex items-center justify-center p-4">
          <div className="bg-white rounded-3xl max-w-lg w-full max-h-[85vh] overflow-y-auto p-6 shadow-2xl space-y-5 animate-in fade-in zoom-in-95 duration-200">
            <div className="flex items-center justify-between border-b border-stone-100 pb-3">
              <div className="flex items-center space-x-2">
                <span className="text-2xl">{selectedLog.mood}</span>
                <span className="text-xs font-semibold text-stone-500">{selectedLog.date}</span>
              </div>
              
              <div className="flex items-center space-x-1">
                <button
                  onClick={() => {
                    openEditForm(selectedLog);
                  }}
                  className="p-1.5 text-stone-400 hover:text-stone-700 hover:bg-stone-100 rounded-lg"
                  title="수정"
                >
                  <Edit className="w-4 h-4" />
                </button>
                <button
                  onClick={() => handleDelete(selectedLog.id)}
                  className="p-1.5 text-stone-400 hover:text-rose-600 hover:bg-rose-50 rounded-lg"
                  title="삭제"
                >
                  <Trash2 className="w-4 h-4" />
                </button>
                <button 
                  onClick={() => setSelectedLog(null)} 
                  className="p-1.5 text-stone-400 hover:text-stone-600 rounded-lg ml-2"
                >
                  <X className="w-5 h-5" />
                </button>
              </div>
            </div>

            <div className="space-y-4">
              <h2 className="text-xl font-bold text-stone-800 leading-snug">
                {selectedLog.title}
              </h2>

              {selectedLog.imageBase64 && (
                <div className="rounded-2xl overflow-hidden border border-stone-100 max-h-72">
                  <img src={selectedLog.imageBase64} alt={selectedLog.title} className="w-full h-full object-cover" />
                </div>
              )}

              <p className="text-stone-700 text-sm leading-relaxed whitespace-pre-wrap font-serif">
                {selectedLog.content}
              </p>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
