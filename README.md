<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>프리미엄 멀티 보험 비교 리포트</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="">
    <link href="https://fonts.googleapis.com/css2?family=Pretendard:wght@400;500;600;700;800&amp;display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js" crossorigin="anonymous"></script>
    <style>
        body { font-family: 'Pretendard', sans-serif; background-color: #f1f5f9; color: #1e293b; }
        
        .slide-wrapper { width: 100%; overflow: hidden; position: relative; }
        /* 모바일 가독성을 위해 min-height를 1200px로 대폭 상향 (글씨 및 비율 조정) */
        .slide-canvas { width: 1200px; min-height: 1200px; background: white; margin: 0 auto; box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.15); border-radius: 8px; overflow: hidden; position: relative; display: flex; flex-direction: column; transform-origin: top left; }
        
        .print-only { display: none; }
        @media print {
            @page { size: A4 landscape; margin: 0; }
            .no-print { display: none !important; }
            .print-only { display: block; }
            body { background: white; padding: 0; margin: 0; }
            .slide-wrapper { height: auto !important; overflow: visible !important; }
            .slide-canvas { 
                transform: none !important; 
                box-shadow: none; 
                margin: 0; 
                border: none; 
                width: 1200px !important; 
                height: 1200px !important; 
                page-break-after: always; 
            }
        }
        input, textarea, select { transition: all 0.2s; }
        input:focus, textarea:focus { border-color: #3b82f6; ring: 2px; ring-color: #bfdbfe; outline: none; }
        .chart-path-old { stroke-dasharray: 8; animation: dash 30s linear infinite; }
        @keyframes dash { to { stroke-dashoffset: -1000; } }

        /* Toast Message */
        #toast { visibility: hidden; min-width: 250px; background-color: #333; color: #fff; text-align: center; border-radius: 12px; padding: 16px; position: fixed; z-index: 100; left: 50%; bottom: 30px; transform: translateX(-50%); font-weight: 600; box-shadow: 0 4px 12px rgba(0,0,0,0.2); }
        #toast.show { visibility: visible; animation: fadein 0.5s, fadeout 0.5s 2.5s; }
        @keyframes fadein { from { bottom: 0; opacity: 0; } to { bottom: 30px; opacity: 1; } }
        @keyframes fadeout { from { bottom: 30px; opacity: 1; } to { bottom: 0; opacity: 0; } }
    
        .disabled-section { opacity: 0.45; pointer-events: none; user-select: none; }
        
        /* Custom Scrollbar for Benefit Inputs */
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
    </style>
</head>
<body class="p-4 md:p-8">
    <div id="toast">메시지가 표시됩니다.</div>
    <div id="scan-review-modal" class="fixed inset-0 bg-slate-900/55 backdrop-blur-sm z-50 hidden items-center justify-center p-4 no-print">
        <div class="w-full max-w-6xl bg-white rounded-3xl shadow-2xl overflow-hidden border border-slate-200">
            <div class="px-6 py-4 border-b border-slate-200 flex items-center justify-between gap-4">
                <div>
                    <h3 class="text-xl font-black text-slate-900">이미지 스캔 검토</h3>
                    <p class="text-sm text-slate-500 mt-1">자동 추출 결과를 적용하기 전에 직접 수정하세요. 향상된 스캔 엔진을 적용했으나 100% 완벽할 수 없습니다.</p>
                </div>
                <button onclick="closeScanReview()" class="w-10 h-10 rounded-full hover:bg-slate-100 text-slate-500"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <div class="p-6 max-h-[75vh] overflow-auto">
                <div class="flex flex-wrap items-center gap-2 mb-4">
                    <button onclick="addScanDraftRow()" class="bg-white border border-slate-200 hover:bg-slate-50 text-slate-700 px-3 py-2 rounded-xl font-bold text-sm">행 추가</button>
                    <button id="btn-scan-append" onclick="applyScanDraft('append')" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-2 rounded-xl font-bold text-sm">추천1 계약에 추가 적용</button>
                    <button id="btn-scan-replace" onclick="applyScanDraft('replace')" class="bg-slate-900 hover:bg-slate-800 text-white px-3 py-2 rounded-xl font-bold text-sm">추천1 계약 전체 교체</button>
                </div>
                <div id="scan-review-table-wrap" class="overflow-auto"></div>
            </div>
        </div>
    </div>

    <!-- 제어 패널 -->
    <div class="max-w-7xl mx-auto mb-12 no-print">
        <header class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 mb-8">
            <div>
                <h1 class="text-3xl font-black text-slate-900 tracking-tight">보험 비교 분석 마스터 <span class="text-blue-600">PRO</span></h1>
                <p class="text-slate-500 mt-1">다수 계약의 복합 분석 및 리모델링 시뮬레이션을 수행합니다.</p>
            </div>
            <div class="flex flex-wrap gap-2">
                <button onclick="resetData()" class="bg-white border border-slate-200 hover:bg-slate-50 text-slate-600 px-4 py-2 rounded-xl font-bold transition flex items-center gap-2">
                    <i class="fa-solid fa-rotate-left"></i> 전체 초기화
                </button>
                <button id="btn-reset-old" onclick="resetOldPremiumsToZero()" class="bg-amber-50 border border-amber-200 hover:bg-amber-100 text-amber-700 px-4 py-2 rounded-xl font-bold transition flex items-center gap-2">
                    <i class="fa-solid fa-coins"></i> 추천1 보험료만 0원 초기화
                </button>
                <button onclick="copyJson()" class="bg-white border border-slate-200 hover:bg-slate-50 text-slate-600 px-4 py-2 rounded-xl font-bold transition flex items-center gap-2">
                    <i class="fa-solid fa-copy"></i> 데이터 복사
                </button>
                <button onclick="saveAllSlides()" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-xl font-bold transition flex items-center gap-2 shadow-lg shadow-blue-100">
                    <i class="fa-solid fa-images"></i> 결과 4장 합쳐서 저장
                </button>
                <button id="btn-apply-old" onclick="applyRenewalToAll()" class="bg-slate-900 hover:bg-slate-800 text-white px-4 py-2 rounded-xl font-bold transition flex items-center gap-2 shadow-lg shadow-slate-200">
                    <i class="fa-solid fa-bolt"></i> 추천1계약 전체 100세납
                </button>
            </div>
        </header>

        <div id="main-data-scope" class="grid grid-cols-1 lg:grid-cols-12 gap-6">
            <div class="lg:col-span-4 space-y-6">
                <section class="bg-white rounded-2xl p-6 shadow-sm border border-slate-100">
                    <h3 class="text-lg font-bold mb-4 flex items-center gap-2"><i class="fa-solid fa-user-tag text-blue-500"></i> 고객 기본 정보</h3>
                    <div class="grid grid-cols-2 md:grid-cols-3 gap-4">
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400">지점</label>
                            <input id="input-branchName" type="text" value="서울중앙지점" readonly="" class="bg-slate-100 border border-slate-200 rounded-lg p-2.5 text-sm font-bold text-slate-700">
                        </div>
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400">담당자 선택 <span class="text-rose-500">*</span></label>
                            <select id="input-managerName" class="bg-white border border-slate-200 rounded-lg p-2.5 text-sm">
                                <option value="">담당자를 선택하세요</option>
                                <option value="선택안함">선택안함</option>
                                <option value="박세민 지점장">박세민 지점장</option>
                                <option value="장규삼 팀장">장규삼 팀장</option>
                                <option value="김병진 팀장">김병진 팀장</option>
                                <option value="박종선 팀장">박종선 팀장</option>
                            </select>
                        </div>
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400">비밀번호 <span class="text-rose-500">*</span></label>
                            <input id="input-managerPassword" type="password" maxlength="4" placeholder="4자리 숫자" class="bg-white border border-slate-200 rounded-lg p-2.5 text-sm">
                        </div>
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400">고객명</label>
                            <input id="input-customerName" type="text" class="bg-slate-50 border border-slate-200 rounded-lg p-2.5 text-sm">
                        </div>
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400">보험나이</label>
                            <input id="input-customerAge" type="number" class="bg-slate-50 border border-slate-200 rounded-lg p-2.5 text-sm">
                        </div>
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400 text-blue-500">비교안 1 명칭</label>
                            <input id="input-planName1" type="text" placeholder="예: 추천1" class="bg-blue-50 border border-blue-200 rounded-lg p-2.5 text-sm font-bold">
                        </div>
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400 text-blue-500">비교안 2 명칭</label>
                            <input id="input-planName2" type="text" placeholder="예: 추천2" class="bg-blue-50 border border-blue-200 rounded-lg p-2.5 text-sm font-bold">
                        </div>
                    </div>
                    <div id="manager-lock-guide" class="mt-3 rounded-xl border border-amber-200 bg-amber-50 px-3 py-2 text-xs text-amber-700 font-medium" style="display: block;">담당자 선택 후 올바른 비밀번호(4자리)를 입력해야 계약 데이터 입력과 이미지 스캔 기능이 활성화됩니다.</div>
                    <div class="mt-4 space-y-4">
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400">핵심 리스크 문구</label>
                            <input id="input-riskMessage" type="text" class="requires-manager bg-slate-50 border border-slate-200 rounded-lg p-2.5 text-sm" disabled="">
                        </div>
                        <div class="flex flex-col gap-1">
                            <label class="text-xs font-bold text-slate-400">요약 코멘트 (1P)</label>
                            <textarea id="input-comment1" class="requires-manager bg-slate-50 border border-slate-200 rounded-lg p-2.5 text-sm h-20" disabled=""></textarea>
                        </div>
                    </div>
                </section>
                <section id="summary-metrics" class="grid grid-cols-1 gap-4">
                    <div class="p-4 rounded-2xl border bg-white shadow-sm">
                        <p class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1">추천1 월 합계</p>
                        <p class="text-2xl font-black text-slate-900">0원</p>
                        <p class="text-xs text-slate-500 mt-1">활성 0건</p>
                    </div>
                    <div class="p-4 rounded-2xl border bg-blue-50 border-blue-100 shadow-sm">
                        <p class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1">추천2 월 합계</p>
                        <p class="text-2xl font-black text-slate-900">0원</p>
                        <p class="text-xs text-slate-500 mt-1">활성 0건</p>
                    </div>
                    <div class="p-4 rounded-2xl border bg-emerald-50 border-emerald-100 shadow-sm">
                        <p class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1">비교 결과 (총액 기준)</p>
                        <p class="text-2xl font-black text-slate-900">0만원</p>
                        <p class="text-xs text-slate-500 mt-1">추천2안이 더 경제적</p>
                    </div>
                </section>
            </div>

            <div class="lg:col-span-8 space-y-6">
                <section class="bg-white rounded-2xl p-6 shadow-sm border border-slate-100">
                    <div class="flex flex-col gap-4 mb-4">
                        <div class="flex justify-between items-center">
                            <h3 id="panel-title-old" class="text-lg font-bold flex items-center gap-2"><i class="fa-solid fa-clock-rotate-left text-orange-500"></i> 추천1 계약 리스트</h3>
                            <button onclick="addPlan('old')" class="requires-manager text-blue-600 hover:text-blue-700 font-bold text-sm" disabled="">+ 추가하기</button>
                        </div>
                        <div class="rounded-2xl border border-dashed border-blue-200 bg-blue-50/50 p-4">
                            <div class="flex flex-col md:flex-row md:items-center md:justify-between gap-3">
                                <div>
                                    <p class="text-sm font-bold text-slate-800">보유계약 리스트 이미지 스캔 가져오기 <span class="text-[10px] bg-blue-100 text-blue-600 px-2 py-0.5 rounded-full ml-1">AI 향상됨</span></p>
                                    <p class="text-xs text-slate-500 mt-1">사전 처리 엔진(스케일업 및 대비 강화)을 적용하여 흐릿한 캡처의 인식률을 대폭 높였습니다.</p>
                                </div>
                                <div class="flex flex-wrap gap-2">
                                    <input id="scan-image-input" type="file" accept="image/*" class="hidden" onchange="handleScanImage(event)">
                                    <button onclick="document.getElementById('scan-image-input').click()" class="requires-manager bg-slate-900 hover:bg-slate-800 text-white px-4 py-2 rounded-xl font-bold text-sm flex items-center gap-2" disabled="">
                                        <i class="fa-solid fa-image"></i> 이미지 선택
                                    </button>
                                    <button onclick="openScanReview()" class="requires-manager bg-white border border-slate-200 hover:bg-slate-50 text-slate-700 px-4 py-2 rounded-xl font-bold text-sm flex items-center gap-2" disabled="">
                                        <i class="fa-solid fa-pen-to-square"></i> 추출결과 검토
                                    </button>
                                </div>
                            </div>
                            <div id="scan-status" class="mt-3 text-xs text-slate-500">담당자 및 비밀번호 인증 전에는 스캔과 데이터 편집이 잠겨 있습니다.</div>
                        </div>
                    </div>
                    <div id="list-oldPlans" class="space-y-4"></div>
                </section>
                
                <!-- 추천 계약 리스트 입력 영역 추가 -->
                <section class="bg-white rounded-2xl p-6 shadow-sm border border-blue-100/50">
                    <div class="flex justify-between items-center mb-4">
                        <h3 id="panel-title-new" class="text-lg font-bold flex items-center gap-2 text-blue-800"><i class="fa-solid fa-hand-holding-heart text-blue-500"></i> 추천2 계약 리스트</h3>
                        <button onclick="addPlan('new')" class="requires-manager text-blue-600 hover:text-blue-700 font-bold text-sm" disabled="">+ 추가하기</button>
                    </div>
                    <div id="list-newPlans" class="space-y-4"></div>
                </section>

                <section class="bg-white rounded-2xl p-6 shadow-sm border border-slate-100">
                    <div class="flex justify-between items-center mb-4">
                        <h3 class="text-lg font-bold flex items-center gap-2"><i class="fa-solid fa-list-check text-emerald-500"></i> 보장 항목 상세 비교 (5대 항목)</h3>
                        <button onclick="resetBenefitsToZero()" class="requires-manager text-slate-500 hover:text-rose-600 font-bold text-sm transition flex items-center gap-1.5" disabled=""><i class="fa-solid fa-rotate-left"></i> 양쪽 0원 초기화</button>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="space-y-4">
                            <p id="panel-benefit-old" class="text-sm font-bold text-slate-400 uppercase tracking-wider">추천1 보장</p>
                            <div class="grid grid-cols-2 gap-3">
                                <div class="flex flex-col gap-1"><label class="text-xs font-bold">암 보장</label><div id="container-benefit-oldCancer"></div></div>
                                <div class="flex flex-col gap-1"><label class="text-xs font-bold">뇌 보장</label><div id="container-benefit-oldBrain"></div></div>
                                <div class="flex flex-col gap-1"><label class="text-xs font-bold">심장 보장</label><div id="container-benefit-oldHeart"></div></div>
                                <div class="flex flex-col gap-1"><label class="text-xs font-bold">수술 보장</label><textarea id="benefit-oldSurgery" class="requires-manager text-[11px] leading-tight p-2 border rounded-lg h-[210px] bg-slate-50" disabled=""></textarea></div>
                                <div class="flex flex-col gap-1 col-span-2"><label class="text-xs font-bold">기타 보장</label><textarea id="benefit-oldOther" class="requires-manager text-[11px] leading-tight p-2 border rounded-lg h-20 bg-slate-50" disabled=""></textarea></div>
                            </div>
                        </div>
                        <div class="space-y-4 border-l pl-0 md:pl-6 border-slate-100">
                            <p id="panel-benefit-new" class="text-sm font-bold text-blue-500 uppercase tracking-wider">추천2 보장</p>
                            <div class="grid grid-cols-2 gap-3">
                                <div class="flex flex-col gap-1"><label class="text-xs font-bold">암 보장</label><div id="container-benefit-newCancer"></div></div>
                                <div class="flex flex-col gap-1"><label class="text-xs font-bold">뇌 보장</label><div id="container-benefit-newBrain"></div></div>
                                <div class="flex flex-col gap-1"><label class="text-xs font-bold">심장 보장</label><div id="container-benefit-newHeart"></div></div>
                                <div class="flex flex-col gap-1"><label class="text-xs font-bold">수술 보장</label><textarea id="benefit-newSurgery" class="requires-manager text-[11px] leading-tight p-2 border border-blue-100 rounded-lg h-[210px] bg-blue-50/30" disabled=""></textarea></div>
                                <div class="flex flex-col gap-1 col-span-2"><label class="text-xs font-bold">기타 보장</label><textarea id="benefit-newOther" class="requires-manager text-[11px] leading-tight p-2 border border-blue-100 rounded-lg h-20 bg-blue-50/30" disabled=""></textarea></div>
                            </div>
                        </div>
                    </div>
                </section>
            </div>
        </div>
    </div>

    <!-- 출력 슬라이드 영역 -->
    <div class="space-y-12 pb-20 no-print">
        
        <!-- 슬라이드 1: 분석 및 진단 -->
        <div class="slide-wrapper relative mx-auto" style="height: 1200px;">
            <div id="slide-1" class="slide-canvas p-10 origin-top-left mx-auto" style="transform: scale(1);">
                <button onclick="saveSlide('slide-1', '리모델링_구조_분석')" class="no-print absolute top-6 right-6 bg-white/80 backdrop-blur border border-slate-200 p-2 px-4 rounded-full text-xs font-bold shadow-sm hover:bg-white transition flex items-center gap-2 z-10">
                    <i class="fa-solid fa-download"></i> PNG 저장
                </button>
                <div class="flex justify-between items-end border-b-2 border-slate-900 pb-5 mb-10">
                    <div>
                        <h2 class="text-5xl font-black text-slate-900 flex items-baseline gap-4">
                            <span onclick="exportFullHtml()" class="text-slate-300 text-4xl font-black italic leading-none cursor-pointer hover:text-slate-400 transition">01</span>
                            보험 구조 정밀 분석 및 리스크 진단
                        </h2>
                    </div>
                    <div class="text-right text-lg font-bold leading-tight text-slate-500" id="slide-user-tag-1"><div>홍길동 (세)</div><div class="font-black text-slate-600">서울중앙지점 · 선택안함</div></div>
                </div>

                <div class="flex flex-col gap-8 flex-grow overflow-hidden">
                    <div class="space-y-4">
                        <h4 id="slide-1-old-table-title" class="text-lg font-black text-slate-500 flex items-center gap-2">
                            <span class="w-3 h-3 rounded-full bg-slate-400"></span> [제안] 추천1 계약 구성 내역
                        </h4>
                        <table class="w-full text-base border-t-2 border-slate-300">
                            <thead>
                                <tr class="bg-slate-50 text-slate-600 border-b border-slate-300">
                                    <th class="py-4 px-3 text-left font-bold">보험사 / 상품명</th>
                                    <th class="py-4 px-3 text-center font-bold">구분</th>
                                    <th class="py-4 px-3 text-center font-bold">가입시기</th>
                                    <th class="py-4 px-3 text-center font-bold">납입/만기</th>
                                    <th class="py-4 px-3 text-right font-bold">월 보험료</th>
                                    <th class="py-4 px-3 text-right font-bold">총 예상 지출액</th>
                                    <th class="py-4 px-3 text-center font-bold">최종 전략</th>
                                </tr>
                            </thead>
                            <tbody id="slide-1-old-table"><tr><td colspan="7" class="text-center py-8 text-lg text-slate-400">데이터가 없습니다.</td></tr></tbody>
                        </table>
                    </div>

                    <div class="space-y-4 mt-4">
                        <h4 id="slide-1-new-table-title" class="text-lg font-black text-blue-500 flex items-center gap-2">
                            <span class="w-3 h-3 rounded-full bg-blue-400"></span> [제안] 추천2 계약 구성 내역
                        </h4>
                        <table class="w-full text-base border-t-2 border-blue-200">
                            <thead>
                                <tr class="bg-blue-50 text-blue-700 border-b border-blue-200">
                                    <th class="py-4 px-3 text-left font-bold">보험사 / 상품명</th>
                                    <th class="py-4 px-3 text-center font-bold">구분</th>
                                    <th class="py-4 px-3 text-center font-bold">납입/만기</th>
                                    <th class="py-4 px-3 text-right font-bold">월 보험료</th>
                                    <th class="py-4 px-3 text-right font-bold">총 납입액</th>
                                    <th class="py-4 px-3 text-center font-bold">전략</th>
                                </tr>
                            </thead>
                            <tbody id="slide-1-new-table"><tr><td colspan="6" class="text-center py-8 text-lg text-slate-400">추천2 계약이 없습니다.</td></tr></tbody>
                        </table>
                    </div>

                    <div class="bg-rose-50 border-l-[8px] border-rose-500 p-6 rounded-r-2xl flex gap-5 items-start mt-4">
                        <i class="fa-solid fa-triangle-exclamation text-rose-500 text-3xl mt-1"></i>
                        <div>
                            <p class="text-rose-900 font-black mb-2 text-lg">핵심 취약점 리스크 체크</p>
                            <p id="slide-1-risk" class="text-rose-800 text-lg leading-relaxed font-medium">월 보험료는 낮아 보이나 암·뇌·심 핵심 보장이 약하고 수술 범위가 좁아 중대 질환 발생 시 실질적인 경제적 방어가 불가능한 구조입니다.</p>
                        </div>
                    </div>

                    <div class="grid grid-cols-2 gap-8 mt-auto pt-6">
                        <div class="bg-slate-50 rounded-[2rem] p-10 border border-slate-200 flex flex-col items-center text-center">
                            <p id="slide-1-old-box-title" class="text-slate-500 text-lg font-bold uppercase mb-4">추천1 구조 유지 시 완납까지</p>
                            <p id="slide-1-old-total" class="text-[4rem] font-black text-slate-900 tracking-tighter mb-4 leading-none">0만원</p>
                            <p class="text-slate-500 text-base font-medium leading-relaxed" id="slide-1-old-sub">월 0원 기준 (100세납 적용) | 완납 나이: <strong>0세</strong></p>
                        </div>
                        <div class="bg-blue-600 rounded-[2rem] p-10 text-white flex flex-col items-center text-center relative overflow-hidden shadow-2xl shadow-blue-100">
                            <div class="absolute -right-8 -top-8 bg-white/10 w-36 h-36 rounded-full"></div>
                            <p id="slide-1-new-box-title" class="text-blue-100 text-lg font-bold uppercase mb-4">추천2 구조 적용 시 완납까지</p>
                            <p id="slide-1-new-total" class="text-[4rem] font-black tracking-tighter mb-4 leading-none">0만원</p>
                            <p class="text-blue-100 text-base font-medium leading-relaxed" id="slide-1-new-sub">월 0원 기준 (100세납 적용) | 완납 나이: <strong>0세</strong></p>
                        </div>
                    </div>
                </div>
                
                <footer class="mt-10 pt-5 border-t border-slate-200 flex justify-between items-center text-[14px] font-bold text-slate-400">
                    <p>* 분석 결과는 입력된 데이터를 기초로 산출되었으며, 갱신형의 경우 100세 납입을 전제로 시뮬레이션 되었습니다.</p>
                    <p>© Premium Insurance Analysis Report</p>
                </footer>
            </div>
        </div>

        <!-- 슬라이드 2: 차트 시뮬레이션 -->
        <div class="slide-wrapper relative mx-auto" style="height: 1200px;">
            <div id="slide-2" class="slide-canvas p-10 origin-top-left mx-auto" style="transform: scale(1);">
                <button onclick="saveSlide('slide-2', '납입_시뮬레이션_차트')" class="no-print absolute top-6 right-6 bg-white/80 backdrop-blur border border-slate-200 p-2 px-4 rounded-full text-xs font-bold shadow-sm hover:bg-white transition flex items-center gap-2 z-10">
                    <i class="fa-solid fa-download"></i> PNG 저장
                </button>
                <div class="flex justify-between items-end border-b-2 border-slate-900 pb-5 mb-10">
                    <div>
                        <h2 class="text-5xl font-black text-slate-900 flex items-baseline gap-4">
                            <span class="text-slate-300 text-4xl font-black italic leading-none">02</span>
                            보험사별 완납 시점 및 누적 지출 비교
                        </h2>
                    </div>
                    <div class="text-right text-lg font-bold leading-tight text-slate-500" id="slide-user-tag-2"><div>홍길동 (세)</div><div class="font-black text-slate-600">서울중앙지점 · 선택안함</div></div>
                </div>
                <div class="flex-grow flex flex-col">
                    <div class="flex justify-center gap-12 mb-8">
                        <div id="slide-2-legend-old" class="flex items-center gap-3 text-[18px] font-bold text-slate-600"><span class="w-5 h-5 bg-slate-300 rounded-full"></span> 추천1 유지 (갱신형 100세납 포함)</div>
                        <div id="slide-2-legend-new" class="flex items-center gap-3 text-[18px] font-bold text-blue-600"><span class="w-5 h-5 bg-blue-600 rounded-full"></span> 추천2 적용</div>
                    </div>
                    <!-- 차트 영역 확대 -->
                    <div class="bg-slate-50 border border-slate-200 rounded-[2rem] p-10 flex-grow">
                        <svg id="svg-chart" viewBox="0 0 1000 550" class="w-full h-full">
                            <!-- SVG contents are dynamically rendered via JavaScript -->
                        </svg>
                    </div>
                    <div class="grid grid-cols-2 gap-10 mt-12">
                        <div class="border-l-[8px] border-blue-600 pl-6 py-2">
                            <h5 class="text-base font-black uppercase tracking-wider text-slate-400 mb-2">총 지출 비용 변화</h5>
                            <p id="slide-2-summary-cost" class="text-slate-800 text-2xl font-bold">총 지출 예상액 <strong class="text-blue-600 text-3xl ml-2">0만원 절감</strong> 가능</p>
                        </div>
                        <div class="border-l-[8px] border-orange-500 pl-6 py-2">
                            <h5 class="text-base font-black uppercase tracking-wider text-slate-400 mb-2">납입 종료 시점 변화</h5>
                            <p id="slide-2-summary-age" class="text-slate-800 text-2xl font-bold">추천1 0세까지의 납입 기간을 <strong class="text-slate-900 text-3xl ml-2">0세 완납</strong> 구조로 변경</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- 슬라이드 3: 보장 비교 -->
        <div class="slide-wrapper relative mx-auto" style="height: 1200px;">
            <div id="slide-3" class="slide-canvas p-10 origin-top-left mx-auto" style="transform: scale(1);">
                <button onclick="saveSlide('slide-3', '보장_상세_비교')" class="no-print absolute top-6 right-6 bg-white/80 backdrop-blur border border-slate-200 p-2 px-4 rounded-full text-xs font-bold shadow-sm hover:bg-white transition flex items-center gap-2 z-10">
                    <i class="fa-solid fa-download"></i> PNG 저장
                </button>
                <div class="flex justify-between items-end border-b-2 border-slate-900 pb-5 mb-10">
                    <div>
                        <h2 class="text-5xl font-black text-slate-900 flex items-baseline gap-4">
                            <span class="text-slate-300 text-4xl font-black italic leading-none">03</span>
                            보장 구조 리모델링 제안
                        </h2>
                    </div>
                    <div class="text-right text-lg font-bold leading-tight text-slate-500" id="slide-user-tag-3"><div>홍길동 (세)</div><div class="font-black text-slate-600">서울중앙지점 · 선택안함</div></div>
                </div>
                <div class="grid grid-cols-2 gap-10 flex-grow items-stretch">
                    <div class="border-2 border-slate-200 rounded-[2rem] overflow-hidden flex flex-col">
                        <div class="bg-slate-100 p-6 text-center border-b border-slate-200"><h4 id="slide-3-old-title" class="text-3xl font-black text-slate-700">추천1 보험</h4></div>
                        <div id="slide-3-old-benefits" class="p-8 space-y-6 flex-grow bg-slate-50/30">
                            <!-- Dynamic Content -->
                        </div>
                    </div>
                    <div class="border-2 border-blue-200 rounded-[2rem] overflow-hidden flex flex-col shadow-2xl shadow-blue-100 relative">
                        <div class="bg-blue-600 p-6 text-center"><h4 id="slide-3-new-title" class="text-3xl font-black text-white">추천2 보험</h4></div>
                        <div id="slide-3-new-benefits" class="p-8 space-y-6 flex-grow bg-blue-50/20">
                            <!-- Dynamic Content -->
                        </div>
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-10 mt-10">
                    <div class="bg-slate-900 text-white rounded-[2rem] p-8"><p id="slide-3-comment-left" class="text-xl font-medium leading-relaxed text-slate-200">가성비 위주의 가입이었으나 실제 보장 한도가 낮고, 만기가 짧아 고령기 의료비 공백이 우려됩니다.</p></div>
                    <div class="bg-blue-600 text-white rounded-[2rem] p-8"><p id="slide-3-comment-right" class="text-xl font-medium leading-relaxed text-blue-50">월 보험료 0원 내에서 생애 전반의 리스크를 커버하도록 구성되었습니다.</p></div>
                </div>
            </div>
        </div>

        <!-- 슬라이드 4: 암 보장 시뮬레이션 -->
        <div class="slide-wrapper relative mx-auto" style="height: 1200px;">
            <div id="slide-4" class="slide-canvas p-10 origin-top-left mx-auto" style="transform: scale(1);">
                <button onclick="saveSlide('slide-4', '암보장_시뮬레이션')" class="no-print absolute top-6 right-6 bg-white/80 backdrop-blur border border-slate-200 p-2 px-4 rounded-full text-xs font-bold shadow-sm hover:bg-white transition flex items-center gap-2 z-10">
                    <i class="fa-solid fa-download"></i> PNG 저장
                </button>
                <div class="flex justify-between items-end border-b-2 border-slate-900 pb-5 mb-10">
                    <div>
                        <h2 class="text-5xl font-black text-slate-900 flex items-baseline gap-4">
                            <span class="text-slate-300 text-4xl font-black italic leading-none">04</span>
                            [시뮬레이션] 주요 암 치료 보상 예시
                        </h2>
                    </div>
                    <div class="text-right text-lg font-bold leading-tight text-slate-500" id="slide-user-tag-4"><div>홍길동 (세)</div><div class="font-black text-slate-600">서울중앙지점 · 선택안함</div></div>
                </div>

                <div class="flex-grow flex flex-col gap-6">
                    <div class="bg-rose-50 border-l-[8px] border-rose-500 text-rose-800 p-5 rounded-r-2xl text-lg font-bold flex items-center gap-4 shadow-sm">
                        <i class="fa-solid fa-bed-pulse text-3xl text-rose-500"></i>
                        <div>
                            <p class="text-rose-900 font-black mb-1">최신 비급여 치료(표적/면역항암 등) 3년 시나리오</p>
                            <p class="text-base font-medium text-rose-700">고액의 비용이 발생하는 비급여 항암 및 방사선 치료를 지속적으로 받을 때의 보상 한도를 비교합니다.</p>
                        </div>
                    </div>

                    <!-- 테이블 -->
                    <div class="border-2 border-slate-200 rounded-[2rem] overflow-hidden flex flex-col flex-grow shadow-sm">
                        <!-- 헤더 -->
                        <div class="grid grid-cols-12 bg-slate-100 border-b-2 border-slate-200 text-center">
                            <div class="col-span-4 p-5 font-black text-xl text-slate-700 border-r border-slate-200">치료 단계 및 시나리오</div>
                            <div id="slide-4-col-old" class="col-span-4 p-5 font-black text-2xl text-slate-500 border-r border-slate-200">추천1 보험</div>
                            <div id="slide-4-col-new" class="col-span-4 p-5 font-black text-2xl text-blue-700 bg-blue-100/50">추천2 보험</div>
                        </div>

                        <!-- 1차년도 -->
                        <div class="grid grid-cols-12 border-b border-slate-200 items-stretch bg-white">
                            <div class="col-span-4 p-6 border-r border-slate-200 flex flex-col justify-center">
                                <div class="inline-block bg-slate-800 text-white text-sm font-bold px-3 py-1 rounded-full mb-3 w-max">1차년도</div>
                                <p class="font-bold text-slate-800 text-xl mb-2">암 최초 진단 및 초기 집중 치료</p>
                            </div>
                            <div class="col-span-4 p-6 border-r border-slate-200 flex flex-col justify-center bg-slate-50/50" id="slide-4-old-y1"></div>
                            <div class="col-span-4 p-6 flex flex-col justify-center bg-blue-50/30" id="slide-4-new-y1"></div>
                        </div>

                        <!-- 2차년도 -->
                        <div class="grid grid-cols-12 border-b border-slate-200 items-stretch bg-white">
                            <div class="col-span-4 p-6 border-r border-slate-200 flex flex-col justify-center">
                                <div class="inline-block bg-slate-800 text-white text-sm font-bold px-3 py-1 rounded-full mb-3 w-max">2차년도</div>
                                <p class="font-bold text-slate-800 text-xl mb-2">본격적인 항암 및 방사선 치료</p>
                            </div>
                            <div class="col-span-4 p-6 border-r border-slate-200 flex flex-col justify-center bg-slate-50/50" id="slide-4-old-y2"></div>
                            <div class="col-span-4 p-6 flex flex-col justify-center bg-blue-50/30" id="slide-4-new-y2"></div>
                        </div>

                        <!-- 3차년도 -->
                        <div class="grid grid-cols-12 border-b border-slate-200 items-stretch bg-white">
                            <div class="col-span-4 p-6 border-r border-slate-200 flex flex-col justify-center">
                                <div class="inline-block bg-slate-800 text-white text-sm font-bold px-3 py-1 rounded-full mb-3 w-max">3차년도</div>
                                <p class="font-bold text-slate-800 text-xl mb-2">유지 및 추적 관찰</p>
                            </div>
                            <div class="col-span-4 p-6 border-r border-slate-200 flex flex-col justify-center bg-slate-50/50" id="slide-4-old-y3"></div>
                            <div class="col-span-4 p-6 flex flex-col justify-center bg-blue-50/30" id="slide-4-new-y3"></div>
                        </div>

                        <!-- 합계 -->
                        <div class="grid grid-cols-12 bg-slate-100 items-stretch flex-grow">
                            <div class="col-span-4 p-6 border-r border-slate-200 flex flex-col justify-center text-right pr-8">
                                <p class="font-black text-slate-700 text-2xl">3년간 누적 보상 예상액</p>
                            </div>
                            <div class="col-span-4 p-6 border-r border-slate-200 flex flex-col justify-center items-center">
                                <p id="slide-4-old-total" class="font-black text-slate-500 text-4xl tracking-tight">0만원</p>
                            </div>
                            <div class="col-span-4 p-6 flex flex-col justify-center items-center bg-blue-600 text-white shadow-inner">
                                <p id="slide-4-new-total" class="font-black text-5xl tracking-tight">0만원</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

<script>
// --- State & Logic ---
const INITIAL_DATA = {
    branchName: '서울중앙지점',
    managerName: '',
    managerPassword: '',
    customerName: '홍길동',
    customerAge: 34,
    planName1: '추천1',
    planName2: '추천2',
    riskMessage: '월 보험료는 낮아 보이나 암·뇌·심 핵심 보장이 약하고 수술 범위가 좁아 중대 질환 발생 시 실질적인 경제적 방어가 불가능한 구조입니다.',
    comment1: '가성비 위주의 가입이었으나 실제 보장 한도가 낮고, 만기가 짧아 고령기 의료비 공백이 우려됩니다.',
    oldPlans: [
        { active: true, company: '메리츠화재', product: '알뜰한건강보험(갱신형)', startDate: '2023.01', maturity: '100세', monthly: 85000, remainingCount: 320, total: '', decision: '해지', renewalCycle: 3, increaseRate: 15 },
        { active: true, company: '삼성생명', product: '종신보험', startDate: '2015.05', maturity: '종신', monthly: 125000, remainingCount: 120, total: 15000000, decision: '유지', renewalCycle: 0, increaseRate: 0 }
    ],
    newPlans: [
        { active: true, company: '현대해상', product: '굿앤굿종합플랜(비갱신)', startDate: '2024.05', maturity: '100세', monthly: 115000, remainingCount: 240, total: 27600000, decision: '신규', renewalCycle: 0 },
        { active: true, company: 'DB손보', product: '실손전환플랜', startDate: '2024.05', maturity: '100세', monthly: 24000, remainingCount: 600, total: 14400000, decision: '신규', renewalCycle: 1, increaseRate: 5 }
    ],
    benefits: {
        oldCancer: { cancerDiag: '0만원', cancerHeavy: '0만원', cancerAnnMax: '0만원', cancerTreatDiag: '0만원', cancerSurg: '0만원', cancerNonSurg: '0만원', cancerNonAnti: '0만원', cancerNonRad: '0만원', cancerCustom: '' },
        oldBrain: { brainVessel: '0만원', brainStroke: '0만원', brainHemorrhage: '0만원', brainCirc: '0만원', brainCustom: '' },
        oldHeart: { heartIschemic: '0만원', heartMyocardial: '0만원', heartCirc: '0만원', heartCustom: '' },
        oldSurgery: '질병수술비 [ 0만원 ]\n상해수술비 [ 0만원 ]',
        oldOther: '가족일상생활배상책임 [ 0만원 ]\n운전자/상해 특약 [ 0만원 ]',
        newCancer: { cancerDiag: '5,000만원', cancerHeavy: '1,000만원', cancerAnnMax: '1억원', cancerTreatDiag: '0만원', cancerSurg: '1,000만원', cancerNonSurg: '500만원', cancerNonAnti: '3,000만원', cancerNonRad: '3,000만원', cancerCustom: '' },
        newBrain: { brainVessel: '2,000만원', brainStroke: '3,000만원', brainHemorrhage: '3,000만원', brainCirc: '2,000만원', brainCustom: '' },
        newHeart: { heartIschemic: '2,000만원', heartMyocardial: '3,000만원', heartCirc: '2,000만원', heartCustom: '' },
        newSurgery: '질병 1-5종 수술비 [ 1,000만원 ]\n상해 1-5종 수술비 [ 1,000만원 ]',
        newOther: '가족일상생활배상책임 1억\n운전자/상해 보장 강화'
    }
};

let state = {
  "branchName": "서울중앙지점",
  "managerName": "선택안함",
  "managerPassword": "",
  "customerName": "홍길동",
  "customerAge": 0,
  "planName1": "추천1",
  "planName2": "추천2",
  "riskMessage": "월 보험료는 낮아 보이나 암·뇌·심 핵심 보장이 약하고 수술 범위가 좁아 중대 질환 발생 시 실질적인 경제적 방어가 불가능한 구조입니다.",
  "comment1": "가성비 위주의 가입이었으나 실제 보장 한도가 낮고, 만기가 짧아 고령기 의료비 공백이 우려됩니다.",
  "oldPlans": [],
  "newPlans": [],
  "benefits": {
    "oldCancer": { "cancerDiag": "0만원", "cancerHeavy": "0만원", "cancerAnnMax": "0만원", "cancerTreatDiag": "0만원", "cancerSurg": "0만원", "cancerNonSurg": "0만원", "cancerNonAnti": "0만원", "cancerNonRad": "0만원", "cancerCustom": "" },
    "oldBrain": { "brainVessel": "0만원", "brainStroke": "0만원", "brainHemorrhage": "0만원", "brainCirc": "0만원", "brainCustom": "" },
    "oldHeart": { "heartIschemic": "0만원", "heartMyocardial": "0만원", "heartCirc": "0만원", "heartCustom": "" },
    "oldSurgery": "질병수술비 [ 0만원 ]\n상해수술비 [ 0만원 ]",
    "oldOther": "가족일상생활배상책임 [ 0만원 ]\n운전자/상해 특약 [ 0만원 ]",
    "newCancer": { "cancerDiag": "5,000만원", "cancerHeavy": "1,000만원", "cancerAnnMax": "1억원", "cancerTreatDiag": "0만원", "cancerSurg": "1,000만원", "cancerNonSurg": "500만원", "cancerNonAnti": "3,000만원", "cancerNonRad": "3,000만원", "cancerCustom": "" },
    "newBrain": { "brainVessel": "2,000만원", "brainStroke": "3,000만원", "brainHemorrhage": "3,000만원", "brainCirc": "2,000만원", "brainCustom": "" },
    "newHeart": { "heartIschemic": "2,000만원", "heartMyocardial": "3,000만원", "heartCirc": "2,000만원", "heartCustom": "" },
    "newSurgery": "질병 1-5종 수술비 [ 1,000만원 ]\n상해 1-5종 수술비 [ 1,000만원 ]",
    "newOther": "가족일상생활배상책임 1억\n운전자/상해 보장 강화"
  }
};
let scanDraftRows = [];
let lastScanImageName = '';

function normalizeOcrText(text) {
    return String(text || '')
        .replace(/[|｜¦]/g, ' ')
        .replace(/[\t]+/g, ' ')
        .replace(/\u00a0/g, ' ')
        .replace(/[ ]{2,}/g, ' ')
        .replace(/\n{2,}/g, '\n')
        .trim();
}

function parseMoneyToken(token) {
    const cleaned = String(token || '').replace(/[^\d]/g, '');
    return cleaned ? Number(cleaned) : 0;
}

function parseCountToken(token) {
    const m = String(token || '').match(/(\d+)\s*\/\s*(\d+)/);
    if (!m) return 0;
    const paid = Number(m[1] || 0);
    const total = Number(m[2] || 0);
    return Math.max(total - paid, 0);
}

function robustOcrParser(rawText) {
    const text = normalizeOcrText(rawText);
    const lines = text.split(/\n/).map(v => v.trim()).filter(Boolean);
        
    let parsedPlans = [];
    let tablePlans = [];
        
    for (let line of lines) {
        const dateMatches = [...line.matchAll(/\d{4}[./-]\d{2}[./-]\d{2}/g)];
        const moneyMatches = [...line.matchAll(/[\d,]+\s*원/g)];
        if (dateMatches.length >= 1 && moneyMatches.length >= 1) {
            const countMatch = line.match(/\d+\s*\/\s*\d+/);
            const noRemoved = line.replace(/^\d+\s+/, '');
            let head = noRemoved;
            if (dateMatches[0]) {
                const idx = noRemoved.indexOf(dateMatches[0][0]);
                if (idx > -1) head = noRemoved.slice(0, idx).trim();
            }
            const headTokens = head.split(' ').filter(Boolean);
            const company = headTokens[0] || '';
            const product = headTokens.slice(1).join(' ').trim();
                        
            tablePlans.push({
                active: true, company, product,
                startDate: dateMatches[0] ? dateMatches[0][0].replace(/-/g,'.').replace(/\//g,'.') : '',
                maturity: dateMatches[1] ? dateMatches[1][0].replace(/-/g,'.').replace(/\//g,'.') : '',
                monthly: moneyMatches[0] ? parseMoneyToken(moneyMatches[0][0]) : 0,
                remainingCount: countMatch ? parseCountToken(countMatch[0]) : 0,
                total: moneyMatches.length >= 3 ? parseMoneyToken(moneyMatches[moneyMatches.length - 1][0]) : 0,
                decision: '유지', renewalCycle: 0, increaseRate: 0, sourceText: line, confidence: '확인필요'
            });
        }
    }
        
    if (tablePlans.filter(p => p.monthly > 0).length >= 1) { 
         tablePlans.forEach(p => { p.confidence = (p.monthly && p.startDate && p.product) ? '높음' : '확인필요'; });
         return tablePlans;
    }

    let current = null;
    for (let i = 0; i < lines.length; i++) {
        const line = lines[i];
                
        const isProductNamePattern = line.includes('보험') && !line.includes('보험료') && !line.includes('피보험자');
                
        if (isProductNamePattern || line.includes('상태 정상') || line.includes('유지중')) {
            if (current && (current.monthly > 0 || current.startDate)) {
                parsedPlans.push(current);
                current = null;
            }
        }
                
        if (!current) {
            current = { active: true, company: '', product: '', startDate: '', maturity: '', monthly: 0, remainingCount: 0, total: 0, decision: '유지', renewalCycle: 0, increaseRate: 0, sourceText: '', confidence: '확인필요' };
        }
                
        current.sourceText += line + ' ';
                
        if (isProductNamePattern) {
            if(!current.product) current.product = line.replace(/^[\(\[\{]*(무배당|무)[\)\]\}]*/, '').trim();
        }
                
        if (line.includes('보험료') || line.match(/[\d,]+\s*원/)) {
            const moneyStr = line.match(/[\d,]+/g);
            if (moneyStr) {
                const val = parseMoneyToken(moneyStr[moneyStr.length - 1]);
                if (val > 1000 && !current.monthly) current.monthly = val;
            }
        }
                
        const dateMatch = line.match(/\d{4}[./-]\d{2}[./-]\d{2}/);
        if (dateMatch) {
            const d = dateMatch[0].replace(/-/g,'.').replace(/\//g,'.');
            if (line.includes('계약') || !current.startDate) current.startDate = d;
            else if (line.includes('만기') && !current.maturity) current.maturity = d;
        }

        if (current.product && !current.company) {
            const p = current.product;
            if (p.includes('교보')) current.company = '교보생명';
            else if (p.includes('삼성')) current.company = '삼성';
            else if (p.includes('현대') || p.includes('Hi')) current.company = '현대해상';
            else if (p.includes('메리츠')) current.company = '메리츠화재';
            else if (p.includes('DB') || p.includes('동부')) current.company = 'DB손보';
            else if (p.includes('KB') || p.includes('국민')) current.company = 'KB손보';
            else if (p.includes('한화')) current.company = '한화';
            else if (p.includes('농협') || p.includes('NH')) current.company = 'NH농협';
            else if (p.includes('라이나')) current.company = '라이나생명';
            else if (p.includes('AIA')) current.company = 'AIA생명';
        }
    }
        
    if (current && (current.monthly > 0 || current.startDate || current.product)) {
        parsedPlans.push(current);
    }
        
    parsedPlans.forEach(p => {
        p.confidence = (p.monthly > 0 && p.startDate && p.product) ? '높음' : '확인필요';
    });
        
    return parsedPlans;
}

async function preprocessImageForOCR(file) {
    return new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => {
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
                        
            let scale = 2;
            if (img.width > 2000) scale = 1.2;
            else if (img.width > 1000) scale = 1.5;

            canvas.width = img.width * scale;
            canvas.height = img.height * scale;
                        
            ctx.filter = 'grayscale(100%) contrast(150%) brightness(105%)';
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                        
            canvas.toBlob(blob => resolve(blob), 'image/jpeg', 0.95);
        };
        img.onerror = reject;
        img.src = URL.createObjectURL(file);
    });
}

async function handleScanImage(event) {
    if (!ensureManagerSelected()) return;
    const file = event.target.files && event.target.files[0];
    if (!file) return;
    lastScanImageName = file.name;
        
    document.getElementById('scan-status').innerHTML = `<span class="font-bold text-slate-700">${escape(file.name)}</span> 분석 중입니다. 화질 개선 엔진이 가동 중입니다.`;
    showToast('이미지 화질을 개선하여 텍스트를 정밀 추출하고 있습니다.');
        
    try {
        const processedBlob = await preprocessImageForOCR(file);

        const result = await Tesseract.recognize(processedBlob, 'kor+eng', {
            logger: m => {
                if (m.status === 'recognizing text') {
                    const pct = Math.round((m.progress || 0) * 100);
                    document.getElementById('scan-status').textContent = `${file.name} 정밀 스캔 중... ${pct}%`;
                }
            }
        });
        const rawText = result?.data?.text || '';
                
        scanDraftRows = robustOcrParser(rawText);
                
        if (!scanDraftRows.length) {
            document.getElementById('scan-status').textContent = '추출된 행이 없습니다. 원본 이미지를 확인해 주세요.';
            showToast('추출된 데이터가 없습니다.');
            return;
        }
        renderScanReviewTable();
        document.getElementById('scan-status').innerHTML = `<span class="font-bold text-blue-600">AI 전처리 완료.</span> <span class="font-bold text-slate-700">${escape(file.name)}</span>에서 ${scanDraftRows.length}건 후보를 추출했습니다.`;
        openScanReview();
        showToast('정밀 스캔 결과를 불러왔습니다.');
    } catch (e) {
        console.error(e);
        document.getElementById('scan-status').textContent = '이미지 분석에 실패했습니다. 브라우저 메모리 또는 인터넷 연결을 확인해 주세요.';
        showToast('이미지 분석에 실패했습니다.');
    } finally {
        event.target.value = '';
    }
}

function renderScanReviewTable() {
    const wrap = document.getElementById('scan-review-table-wrap');
    if (!scanDraftRows.length) {
        wrap.innerHTML = '<div class="text-sm text-slate-400 py-8 text-center">검토할 추출 결과가 없습니다.</div>';
        return;
    }
    wrap.innerHTML = `
        <table class="w-full min-w-[1100px] text-xs border border-slate-200 rounded-2xl overflow-hidden">
            <thead>
                <tr class="bg-slate-50 text-slate-500">
                    <th class="px-3 py-3 text-left font-bold">회사</th>
                    <th class="px-3 py-3 text-left font-bold">상품명</th>
                    <th class="px-3 py-3 text-left font-bold">계약일</th>
                    <th class="px-3 py-3 text-left font-bold">보장만기</th>
                    <th class="px-3 py-3 text-right font-bold">월 보험료</th>
                    <th class="px-3 py-3 text-right font-bold">잔여 납입횟수</th>
                    <th class="px-3 py-3 text-right font-bold">잔여 보험료</th>
                    <th class="px-3 py-3 text-center font-bold">신뢰도</th>
                    <th class="px-3 py-3 text-center font-bold">삭제</th>
                </tr>
            </thead>
            <tbody>
                ${scanDraftRows.map((row, i) => `
                    <tr class="border-t border-slate-200">
                        <td class="p-2"><input value="${escape(row.company)}" oninput="updateScanDraftRow(${i}, 'company', this.value)" class="requires-manager w-full p-2 border rounded-lg bg-white"></td>
                        <td class="p-2"><input value="${escape(row.product)}" oninput="updateScanDraftRow(${i}, 'product', this.value)" class="requires-manager w-full p-2 border rounded-lg bg-white"></td>
                        <td class="p-2"><input value="${escape(row.startDate)}" oninput="updateScanDraftRow(${i}, 'startDate', this.value)" class="requires-manager w-full p-2 border rounded-lg bg-white"></td>
                        <td class="p-2">${renderMaturityEditor('scan', i, row.maturity || '')}</td>
                        <td class="p-2"><input type="number" value="${Number(row.monthly || 0)}" oninput="updateScanDraftRow(${i}, 'monthly', this.value)" class="requires-manager w-full p-2 border rounded-lg bg-white text-right"></td>
                        <td class="p-2"><input type="number" value="${Number(row.remainingCount || 0)}" oninput="updateScanDraftRow(${i}, 'remainingCount', this.value)" class="requires-manager w-full p-2 border rounded-lg bg-white text-right"></td>
                        <td class="p-2"><input type="number" value="${Number(row.total || 0)}" oninput="updateScanDraftRow(${i}, 'total', this.value)" class="requires-manager w-full p-2 border rounded-lg bg-white text-right"></td>
                        <td class="p-2 text-center"><span class="px-2 py-1 rounded-full font-bold ${row.confidence === '높음' ? 'bg-emerald-50 text-emerald-600' : 'bg-amber-50 text-amber-600'}">${row.confidence}</span></td>
                        <td class="p-2 text-center"><button onclick="removeScanDraftRow(${i})" class="requires-manager text-rose-500 hover:text-rose-700"><i class="fa-solid fa-trash-can"></i></button></td>
                    </tr>
                    <tr class="border-t border-slate-100 bg-slate-50/60">
                        <td colspan="9" class="px-3 py-2 text-[11px] text-slate-500"><span class="font-bold text-slate-700">원문 OCR</span> ${escape(row.sourceText || '')}</td>
                    </tr>
                `).join('')}
            </tbody>
        </table>`;
}

function updateScanDraftRow(idx, key, value) {
    if (!ensureManagerSelected()) return;
    scanDraftRows[idx][key] = ['monthly','remainingCount','total'].includes(key) ? Number(value || 0) : value;
}

function removeScanDraftRow(idx) {
    if (!ensureManagerSelected()) return;
    scanDraftRows.splice(idx, 1);
    renderScanReviewTable();
    syncActivationState();
}

function addScanDraftRow() {
    if (!ensureManagerSelected()) return;
    scanDraftRows.push({ active: true, company: '', product: '', startDate: '', maturity: '', monthly: 0, remainingCount: 0, total: 0, decision: '유지', renewalCycle: 0, increaseRate: 0, sourceText: '수기 추가', confidence: '확인필요' });
    renderScanReviewTable();
    syncActivationState();
}

function openScanReview() {
    if (!ensureManagerSelected()) return;
    const modal = document.getElementById('scan-review-modal');
    renderScanReviewTable();
    modal.classList.remove('hidden');
    modal.classList.add('flex');
}

function closeScanReview() {
    const modal = document.getElementById('scan-review-modal');
    modal.classList.add('hidden');
    modal.classList.remove('flex');
}

function applyScanDraft(mode) {
    if (!ensureManagerSelected()) return;
    const cleaned = scanDraftRows
        .filter(r => (r.company || r.product || Number(r.monthly) > 0))
        .map(r => ({
            active: true,
            company: r.company || '',
            product: r.product || '',
            startDate: r.startDate || '',
            maturity: r.maturity || '',
            monthly: Number(r.monthly || 0),
            remainingCount: Number(r.remainingCount || 0),
            total: Number(r.total || 0),
            decision: '유지',
            renewalCycle: 0,
            increaseRate: 0
        }));
    if (!cleaned.length) {
        showToast('적용할 행이 없습니다.');
        return;
    }
    state.oldPlans = mode === 'replace' ? cleaned : [...state.oldPlans, ...cleaned];
    renderInputLists();
    syncActivationState();
    updateSlidesAndMetrics();
    closeScanReview();
    
    const p1 = state.planName1 || '추천1';
    showToast(mode === 'replace' ? `스캔 결과로 ${p1} 계약을 교체했습니다.` : `스캔 결과를 ${p1} 계약에 추가했습니다.`);
}

const BENEFIT_TEMPLATES = {
    Cancer: [
        { type: 'header', text: '최초 1회' },
        { type: 'field', key: 'cancerDiag', label: '암진단비' },
        { type: 'field', key: 'cancerHeavy', label: '중입자 치료비' },
        { type: 'header', text: '매년 보상' },
        { type: 'field', key: 'cancerAnnMax', label: '암종합치료비 연간 최대' },
        { type: 'field', key: 'cancerTreatDiag', label: '암치료진단비' },
        { type: 'field', key: 'cancerSurg', label: '암수술비' },
        { type: 'field', key: 'cancerNonSurg', label: '비급여수술비' },
        { type: 'field', key: 'cancerNonAnti', label: '비급여항암' },
        { type: 'field', key: 'cancerNonRad', label: '비급여방사선' },
        { type: 'custom', key: 'cancerCustom', label: '직접 입력' }
    ],
    Brain: [
        { type: 'header', text: '최초 1회 진단비' },
        { type: 'field', key: 'brainVessel', label: '뇌혈관(초기~말기)' },
        { type: 'field', key: 'brainStroke', label: '뇌졸중(중기~말기)' },
        { type: 'field', key: 'brainHemorrhage', label: '뇌출혈(말기)' },
        { type: 'header', text: '매년 보상 치료비' },
        { type: 'field', key: 'brainCirc', label: '순환기계 치료비(가장 넓음)' },
        { type: 'custom', key: 'brainCustom', label: '직접 입력' }
    ],
    Heart: [
        { type: 'header', text: '최초 1회 진단비' },
        { type: 'field', key: 'heartIschemic', label: '허혈성심장질환(초기~말기)' },
        { type: 'field', key: 'heartMyocardial', label: '급성심근경색(말기)' },
        { type: 'header', text: '매년 보상 치료비' },
        { type: 'field', key: 'heartCirc', label: '순환기계 치료비(가장 넓음)' },
        { type: 'custom', key: 'heartCustom', label: '직접 입력' }
    ]
};

function renderBenefitForm() {
    const renderStruct = (type, category) => {
        const data = state.benefits[`${type}${category}`];
        const tpl = BENEFIT_TEMPLATES[category];
        const isNew = type === 'new';
        let html = `<div class="bg-${isNew?'blue':'slate'}-50/${isNew?'30':'100'} p-2 border ${isNew?'border-blue-100':'border-slate-200'} rounded-lg h-[210px] overflow-y-auto text-[10px] space-y-1.5 custom-scrollbar">`;
        tpl.forEach(item => {
            if (item.type === 'header') {
                html += `<div class="font-bold text-${isNew?'blue':'slate'}-600 mt-1">${item.text}</div>`;
            } else if (item.type === 'custom') {
                const val = data[item.key] || '';
                html += `<div class="flex items-center mt-2">
                    <input type="text" value="${escape(val)}" placeholder="예: 특별조건 [ 0만원 ]" oninput="updateStructBenefit('${type}', '${category}', '${item.key}', this.value, true)" class="requires-manager w-full p-1.5 border border-slate-200 rounded bg-white text-${isNew?'blue':'slate'}-700 focus:border-blue-500 outline-none transition-all">
                </div>`;
            } else {
                const displayValue = (data[item.key] || '').replace(/만원/g, '').trim();
                html += `<div class="flex items-center justify-between gap-1">
                    <span class="text-slate-600 leading-tight flex-1 tracking-tight">${item.label}</span>
                    <div class="flex items-center gap-1 shrink-0">
                        <input type="text" value="${escape(displayValue)}" oninput="updateStructBenefit('${type}', '${category}', '${item.key}', this.value)" class="requires-manager w-[46px] text-right p-1 border border-slate-200 rounded bg-white text-${isNew?'blue':'slate'}-700 font-bold focus:border-blue-500 focus:ring-1 focus:ring-blue-500 outline-none transition-all">
                        <span class="text-[10px] text-slate-500 font-bold whitespace-nowrap">만원</span>
                    </div>
                </div>`;
            }
        });
        html += `</div>`;
        document.getElementById(`container-benefit-${type}${category}`).innerHTML = html;
    };

    ['old', 'new'].forEach(t => {
        ['Cancer', 'Brain', 'Heart'].forEach(c => renderStruct(t, c));
    });
}

function updateStructBenefit(type, category, key, value, isCustom = false) {
    if (!ensureManagerSelected()) return;
    
    if (isCustom) {
        state.benefits[`${type}${category}`][key] = value;
    } else {
        let cleanVal = value.replace(/만원/g, '').trim();
        if (!cleanVal) cleanVal = '0';
        state.benefits[`${type}${category}`][key] = cleanVal + '만원';
    }
    
    updateSlidesAndMetrics();
}

const fmt = v => Number(v || 0).toLocaleString();
const won = v => fmt(v) + '원';
const man = v => (Math.round(v/10000)).toLocaleString() + '만원';
const escape = s => String(s || '').replace(/[&<>"']/g, m => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]));

function hasManagerSelected() {
    if (state.managerName === '박세민 지점장' && state.managerPassword === '5777') return true;
    if (state.managerName === '장규삼 팀장' && state.managerPassword === '9393') return true;
    if (state.managerName === '김병진 팀장' && state.managerPassword === '2580') return true;
    if (state.managerName === '박종선 팀장' && state.managerPassword === '2518') return true;
    return false;
}

function ensureManagerSelected() {
    if (hasManagerSelected()) return true;
    showToast('담당자 선택 후 올바른 비밀번호를 입력해 주세요.');
    const el = document.getElementById('input-managerPassword');
    if (el) el.focus();
    return false;
}

function getMaturityMode(value) {
    const v = String(value || '').trim();
    return ['100세','90세','80세'].includes(v) ? v : 'custom';
}

function renderMaturityEditor(context, idx, value) {
    const mode = getMaturityMode(value);
    const customValue = mode === 'custom' ? escape(value || '') : '';
    return `
        <div class="grid grid-cols-1 gap-2 sm:grid-cols-[110px_minmax(0,1fr)] sm:items-center">
            <select onchange="changeMaturityMode('${context}', ${idx}, this.value)" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">
                <option value="100세" ${mode === '100세' ? 'selected' : ''}>100세</option>
                <option value="90세" ${mode === '90세' ? 'selected' : ''}>90세</option>
                <option value="80세" ${mode === '80세' ? 'selected' : ''}>80세</option>
                <option value="custom" ${mode === 'custom' ? 'selected' : ''}>수기입력</option>
            </select>
            <input ${mode === 'custom' ? '' : 'disabled'} value="${customValue}" oninput="updateMaturityCustom('${context}', ${idx}, this.value)" placeholder="직접 입력" class="requires-manager h-10 w-full rounded-lg border border-slate-200 px-2 text-xs ${mode === 'custom' ? 'bg-white' : 'bg-slate-100 text-slate-400'}">
        </div>`;
}

function changeMaturityMode(context, idx, mode) {
    if (context === 'scan') {
        scanDraftRows[idx].maturity = mode === 'custom' ? '' : mode;
        renderScanReviewTable();
        return;
    }
    state[`${context}Plans`][idx].maturity = mode === 'custom' ? '' : mode;
    renderInputLists();
    syncActivationState();
    updateSlidesAndMetrics();
}

function updateMaturityCustom(context, idx, value) {
    if (context === 'scan') {
        scanDraftRows[idx].maturity = value;
        return;
    }
    state[`${context}Plans`][idx].maturity = value;
    updateSlidesAndMetrics();
}

function syncActivationState() {
    const enabled = hasManagerSelected();
    document.querySelectorAll('.requires-manager').forEach(el => {
        el.disabled = !enabled;
    });
    const guide = document.getElementById('manager-lock-guide');
    if (guide) guide.style.display = enabled ? 'none' : 'block';
    const scanStatus = document.getElementById('scan-status');
    if (scanStatus && !enabled && !lastScanImageName) {
        scanStatus.textContent = '담당자 및 비밀번호 인증 전에는 스캔과 데이터 편집이 잠겨 있습니다.';
    }
}

function showToast(msg) {
    const t = document.getElementById("toast");
    t.textContent = msg;
    t.className = "show";
    setTimeout(() => { t.className = t.className.replace("show", ""); }, 3000);
}

const isPlanRenewal = p => Number(p.renewalCycle || 0) > 0;

function getActualMonths(p, baseAge) {
    if (isPlanRenewal(p)) return Math.max(0, (100 - Number(baseAge)) * 12);
    return Number(p.remainingCount || 0);
}

function calcCumulativeAtAge(p, baseAge, checkAge) {
    const monthly = Number(p.monthly || 0);
    const cycle = Number(p.renewalCycle || 0);
    const rate = Number(p.increaseRate || 0) / 100;
    const monthsDiff = (checkAge - baseAge) * 12;
    if (monthsDiff <= 0) return 0;
    if (!isPlanRenewal(p)) {
        const totalMonths = Number(p.remainingCount || 0);
        return monthly * Math.min(monthsDiff, totalMonths);
    }
    let currentMonthly = monthly, cumulative = 0;
    const totalRenewalMonths = Math.min(monthsDiff, (100 - baseAge) * 12);
    for (let m = 1; m <= totalRenewalMonths; m++) {
        cumulative += currentMonthly;
        if (m % (cycle * 12) === 0) currentMonthly *= (1 + rate);
    }
    return Math.round(cumulative);
}

function calcPlanTotalProjected(p, baseAge) {
    return calcCumulativeAtAge(p, baseAge, 100);
}

function getStats() {
    const age = Number(state.customerAge);
    const oldActive = state.oldPlans.filter(p => p.active);
    const newActive = state.newPlans.filter(p => p.active);
    const oldMonthly = oldActive.reduce((s, p) => s + Number(p.monthly), 0);
    const oldProjectedTotal = oldActive.reduce((s, p) => s + calcPlanTotalProjected(p, age), 0);
    const oldMaxMonths = oldActive.reduce((m, p) => Math.max(m, getActualMonths(p, age)), 0);
    const newMonthly = newActive.reduce((s, p) => s + Number(p.monthly), 0);
    const newTotal = newActive.reduce((s, p) => s + calcPlanTotalProjected(p, age), 0);
    const newMaxMonths = newActive.reduce((m, p) => Math.max(m, getActualMonths(p, age)), 0);
    return { age, oldMonthly, oldProjectedTotal, oldMaxMonths, newMonthly, newTotal, newMaxMonths, save: oldProjectedTotal - newTotal, oldPaidAge: age + (oldMaxMonths / 12), newPaidAge: age + (newMaxMonths / 12) };
}

function initForm() {
    document.getElementById('input-branchName').value = state.branchName || '서울중앙지점';
    const managerEl = document.getElementById('input-managerName');
    managerEl.value = state.managerName || '';
    managerEl.onchange = (e) => { state.managerName = e.target.value; syncActivationState(); updateSlidesAndMetrics(); };
        
    const pwdEl = document.getElementById('input-managerPassword');
    if (pwdEl) {
        pwdEl.value = state.managerPassword || '';
        pwdEl.oninput = (e) => { state.managerPassword = e.target.value; syncActivationState(); };
    }

    ['customerName', 'customerAge', 'riskMessage', 'comment1', 'planName1', 'planName2'].forEach(key => {
        const el = document.getElementById('input-' + key);
        if(el) {
            el.value = state[key];
            el.oninput = (e) => { state[key] = key === 'customerAge' ? Number(e.target.value) : e.target.value; updateSlidesAndMetrics(); };
        }
    });
    
    ['oldSurgery', 'newSurgery', 'oldOther', 'newOther'].forEach(key => {
        const el = document.getElementById('benefit-' + key);
        if(el) {
            el.value = state.benefits[key];
            el.oninput = (e) => { state.benefits[key] = e.target.value; updateSlidesAndMetrics(); };
        }
    });

    renderBenefitForm();
    renderInputLists();
    syncActivationState();
    updateSlidesAndMetrics();
    window.addEventListener('resize', adjustScale);
}

function renderInputLists() {
    renderPlanInputList('old');
    renderPlanInputList('new');
}

function renderPlanInputList(type) {
    const container = document.getElementById(`list-${type}Plans`);
    const plans = state[`${type}Plans`];
    container.innerHTML = plans.map((p, i) => {
        const isRenewal = isPlanRenewal(p);
        const decisionHtml = type === 'old'
            ? `<select onchange="updatePlanValue('${type}', ${i}, 'decision', this.value)" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">
                <option value="유지" ${p.decision==='유지'?'selected':''}>유지</option>
                <option value="유지검토" ${p.decision==='유지검토'?'selected':''}>유지검토</option>
                <option value="해지" ${p.decision==='해지'?'selected':''}>해지</option>
                <option value="부분해지" ${p.decision==='부분해지'?'selected':''}>부분해지</option>
                <option value="감액완납" ${p.decision==='감액완납'?'selected':''}>감액완납</option>
               </select>`
            : `<input oninput="updatePlanValue('${type}', ${i}, 'decision', this.value)" value="${escape(p.decision)}" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">`;

        return `
        <div class="rounded-2xl border border-slate-200 bg-slate-50 p-4">
            <div class="mb-4 grid grid-cols-1 gap-3 xl:grid-cols-12">
                <div class="min-w-0 xl:col-span-2">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">보험사</label>
                    <input oninput="updatePlanValue('${type}', ${i}, 'company', this.value)" value="${escape(p.company)}" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">
                </div>
                <div class="min-w-0 xl:col-span-3">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">상품명</label>
                    <input oninput="updatePlanValue('${type}', ${i}, 'product', this.value)" value="${escape(p.product)}" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">
                </div>
                <div class="min-w-0 xl:col-span-3">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">보장 만기</label>
                    ${renderMaturityEditor(type, i, p.maturity || '')}
                </div>
                <div class="min-w-0 xl:col-span-2">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">월 보험료</label>
                    <input type="number" oninput="updatePlanValue('${type}', ${i}, 'monthly', this.value)" value="${p.monthly}" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">
                </div>
                <div class="min-w-0 xl:col-span-2">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">납입횟수(비갱신)</label>
                    <input type="number" ${isRenewal ? 'disabled placeholder="100세납 고정"' : ''} oninput="updatePlanValue('${type}', ${i}, 'remainingCount', this.value)" value="${p.remainingCount}" class="requires-manager h-10 w-full rounded-lg border border-slate-200 px-2 text-xs ${isRenewal ? 'bg-slate-100 italic text-slate-400' : 'bg-white'}">
                </div>
            </div>

            <div class="grid grid-cols-1 gap-3 lg:grid-cols-12 lg:items-end">
                <div class="min-w-0 lg:col-span-2">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">총 고정액</label>
                    <input type="number" oninput="updatePlanValue('${type}', ${i}, 'total', this.value)" value="${p.total}" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">
                </div>
                <div class="min-w-0 lg:col-span-3">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">전략/비고</label>
                    ${decisionHtml}
                </div>
                <div class="min-w-0 lg:col-span-2">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">갱신주기(년)</label>
                    <input type="number" oninput="updatePlanValue('${type}', ${i}, 'renewalCycle', this.value)" value="${p.renewalCycle || 0}" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">
                </div>
                <div class="min-w-0 lg:col-span-2">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">갱신 시 인상률(%)</label>
                    <input type="number" oninput="updatePlanValue('${type}', ${i}, 'increaseRate', this.value)" value="${p.increaseRate || 0}" class="requires-manager h-10 w-full rounded-lg border border-slate-200 bg-white px-2 text-xs">
                </div>
                <div class="min-w-0 lg:col-span-2">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">반영 여부</label>
                    <div class="flex h-10 items-center justify-between rounded-lg border border-slate-200 bg-white px-3">
                        <label class="flex items-center gap-2 text-xs font-bold text-slate-600"><input class="requires-manager" type="checkbox" ${p.active?'checked':''} onchange="updatePlanValue('${type}', ${i}, 'active', this.checked)"> 반영</label>
                        <button onclick="removePlan('${type}', ${i})" class="requires-manager p-1 text-rose-500 hover:text-rose-700"><i class="fa-solid fa-trash-can"></i></button>
                    </div>
                </div>
                <div class="min-w-0 lg:col-span-3">
                    <label class="mb-1 block text-[10px] font-bold text-slate-400">예상 총액</label>
                    <div id="est-${type}-${i}" class="flex h-10 items-center rounded-lg border border-orange-200 bg-orange-50 px-3 text-[11px] font-bold text-orange-600">${isRenewal ? '100세 납입 기준 총액' : '납입 완료까지 총액'}: ${won(calcPlanTotalProjected(p, state.customerAge))}</div>
                </div>
            </div>
        </div>`;
    }).join('');
}

function updatePlanValue(type, idx, key, val) {
    if (!ensureManagerSelected()) return;
    state[`${type}Plans`][idx][key] = (['monthly','remainingCount','total','renewalCycle','increaseRate'].includes(key)) ? Number(val) : val;
    const estLabel = document.getElementById(`est-${type}-${idx}`);
    if(estLabel) {
        const p = state[`${type}Plans`][idx];
        estLabel.textContent = `${isPlanRenewal(p) ? '100세 납입 기준 총액' : '납입 완료까지 총액'}: ${won(calcPlanTotalProjected(p, state.customerAge))}`;
    }
    updateSlidesAndMetrics();
}

function addPlan(type) {
    if (!ensureManagerSelected()) return;
    const plan = type === 'old' 
        ? { active: true, company: '', product: '', startDate: '', maturity: '', monthly: 0, remainingCount: 0, total: '', decision: '유지', renewalCycle: 0, increaseRate: 0 }
        : { active: true, company: '', product: '', startDate: '', maturity: '', monthly: 0, remainingCount: 0, total: '', decision: '신규', renewalCycle: 0, increaseRate: 0 };
    state[`${type}Plans`].push(plan);
    renderInputLists();
    syncActivationState();
    updateSlidesAndMetrics();
}

function removePlan(type, idx) {
    if (!ensureManagerSelected()) return;
    state[`${type}Plans`].splice(idx, 1);
    renderInputLists();
    syncActivationState();
    updateSlidesAndMetrics();
}

function adjustScale() {
    document.querySelectorAll('.slide-wrapper').forEach(wrapper => {
        const canvas = wrapper.querySelector('.slide-canvas');
        if (!canvas) return;
        const w = wrapper.clientWidth;
        const scale = w < 1200 ? w / 1200 : 1; 
        canvas.style.transform = `scale(${scale})`;
        wrapper.style.height = `${canvas.offsetHeight * scale}px`;
    });
}

let _slideUpdateTimer = null;
function updateSlidesAndMetrics() {
    if (_slideUpdateTimer) clearTimeout(_slideUpdateTimer);
    _slideUpdateTimer = setTimeout(() => {
        _executeSlideUpdates();
    }, 150); 
}

function _executeSlideUpdates() {
    const s = getStats();
    const p1 = escape(state.planName1 || '추천1');
    const p2 = escape(state.planName2 || '추천2');

    const metrics = [
        { label: `${p1} 월 합계`, val: won(s.oldMonthly), sub: `활성 ${state.oldPlans.filter(p=>p.active).length}건`, cls: 'bg-white' },
        { label: `${p2} 월 합계`, val: won(s.newMonthly), sub: `활성 ${state.newPlans.filter(p=>p.active).length}건`, cls: 'bg-blue-50 border-blue-100' },
        { label: '비교 결과 (총액 기준)', val: man(Math.abs(s.save)), sub: s.save >= 0 ? `${p2}안이 더 경제적` : '보장 확대로 인한 증가', cls: s.save >= 0 ? 'bg-emerald-50 border-emerald-100' : 'bg-rose-50 border-rose-100' }
    ];
    document.getElementById('summary-metrics').innerHTML = metrics.map(m => `
        <div class="p-4 rounded-2xl border ${m.cls} shadow-sm">
            <p class="text-[10px] font-black text-slate-400 uppercase tracking-wider mb-1">${m.label}</p>
            <p class="text-2xl font-black text-slate-900">${m.val}</p>
            <p class="text-xs text-slate-500 mt-1">${m.sub}</p>
        </div>`).join('');

    const userTag = `${state.customerName} (${state.customerAge}세) · ${state.branchName}${state.managerName ? ' · ' + state.managerName : ''}`;
    [1,2,3,4].forEach(n => { 
        const el = document.getElementById(`slide-user-tag-${n}`);
        if(el) el.innerHTML = `<div>${escape(state.customerName)} (${escape(state.customerAge)}세)</div><div class="font-black text-slate-600">${escape(state.branchName)}${state.managerName ? ' · ' + escape(state.managerName) : ''}</div>`; 
    });

    // Update Titles and Buttons with p1/p2
    const p_old_title = document.getElementById('panel-title-old'); if(p_old_title) p_old_title.innerHTML = `<i class="fa-solid fa-clock-rotate-left text-orange-500"></i> ${p1} 계약 리스트`;
    const p_new_title = document.getElementById('panel-title-new'); if(p_new_title) p_new_title.innerHTML = `<i class="fa-solid fa-hand-holding-heart text-blue-500"></i> ${p2} 계약 리스트`;
    
    const p_old_ben = document.getElementById('panel-benefit-old'); if(p_old_ben) p_old_ben.textContent = `${p1} 보장`;
    const p_new_ben = document.getElementById('panel-benefit-new'); if(p_new_ben) p_new_ben.textContent = `${p2} 보장`;

    const btn_reset = document.getElementById('btn-reset-old'); if(btn_reset) btn_reset.innerHTML = `<i class="fa-solid fa-coins"></i> ${p1} 보험료만 0원 초기화`;
    const btn_apply = document.getElementById('btn-apply-old'); if(btn_apply) btn_apply.innerHTML = `<i class="fa-solid fa-bolt"></i> ${p1}계약 전체 100세납`;
    
    const btn_scan_append = document.getElementById('btn-scan-append'); if(btn_scan_append) btn_scan_append.textContent = `${p1} 계약에 추가 적용`;
    const btn_scan_replace = document.getElementById('btn-scan-replace'); if(btn_scan_replace) btn_scan_replace.textContent = `${p1} 계약 전체 교체`;

    const s1_old_t = document.getElementById('slide-1-old-table-title'); if(s1_old_t) s1_old_t.innerHTML = `<span class="w-3 h-3 rounded-full bg-slate-400"></span> [제안] ${p1} 계약 구성 내역`;
    const s1_new_t = document.getElementById('slide-1-new-table-title'); if(s1_new_t) s1_new_t.innerHTML = `<span class="w-3 h-3 rounded-full bg-blue-400"></span> [제안] ${p2} 계약 구성 내역`;
    const s1_old_box = document.getElementById('slide-1-old-box-title'); if(s1_old_box) s1_old_box.textContent = `${p1} 구조 유지 시 완납까지`;
    const s1_new_box = document.getElementById('slide-1-new-box-title'); if(s1_new_box) s1_new_box.textContent = `${p2} 구조 적용 시 완납까지`;

    const oldActive = state.oldPlans.filter(p => p.active);
    document.getElementById('slide-1-old-table').innerHTML = oldActive.length ? oldActive.map(p => `
        <tr class="border-b border-slate-200">
            <td class="py-4 px-3"><strong class="text-[17px]">${escape(p.company)}</strong><br/><span class="text-[14px] text-slate-500 mt-1 inline-block">${escape(p.product)}</span></td>
            <td class="py-4 px-3 text-center text-[14px] font-bold ${isPlanRenewal(p) ? 'text-rose-600' : 'text-slate-500'}">${isPlanRenewal(p) ? '갱신형' : '비갱신형'}</td>
            <td class="py-4 px-3 text-center text-[16px]">${escape(p.startDate || '-')}</td>
            <td class="py-4 px-3 text-center"><strong class="text-[16px]">${isPlanRenewal(p) ? '100세 납입' : fmt(p.remainingCount)+'회'}</strong><br/><span class="text-[14px] text-slate-500 mt-1 inline-block">${escape(p.maturity || '-')} 만기</span></td>
            <td class="py-4 px-3 text-right text-[16px]">${fmt(p.monthly)}원</td>
            <td class="py-4 px-3 text-right text-[18px] font-black ${isPlanRenewal(p) ? 'text-rose-600' : 'text-slate-900'}">${fmt(calcPlanTotalProjected(p, state.customerAge))}원</td>
            <td class="py-4 px-3 text-center font-bold text-slate-600 text-[16px]">${escape(p.decision)}</td>
        </tr>`).join('') : `<tr><td colspan="7" class="text-center py-8 text-lg text-slate-400">${p1} 계약이 없습니다.</td></tr>`;

    const newActive = state.newPlans.filter(p => p.active);
    document.getElementById('slide-1-new-table').innerHTML = newActive.length ? newActive.map(p => `
        <tr class="border-b border-blue-100">
            <td class="py-4 px-3"><strong class="text-[17px]">${escape(p.company)}</strong><br/><span class="text-[14px] text-blue-500 mt-1 inline-block">${escape(p.product)}</span></td>
            <td class="py-4 px-3 text-center text-[14px] font-bold ${isPlanRenewal(p) ? 'text-rose-600' : 'text-blue-600'}">${isPlanRenewal(p) ? '갱신형' : '비갱신형'}</td>
            <td class="py-4 px-3 text-center"><strong class="text-[16px]">${isPlanRenewal(p) ? '100세 납입' : fmt(p.remainingCount)+'회'}</strong><br/><span class="text-[14px] text-blue-500 mt-1 inline-block">${escape(p.maturity || '-')} 만기</span></td>
            <td class="py-4 px-3 text-right text-[16px]">${fmt(p.monthly)}원</td>
            <td class="py-4 px-3 text-right text-[18px] font-black text-blue-700">${fmt(calcPlanTotalProjected(p, state.customerAge))}원</td>
            <td class="py-4 px-3 text-center font-bold text-blue-600 text-[16px]">${escape(p.decision)}</td>
        </tr>`).join('') : `<tr><td colspan="6" class="text-center py-8 text-lg text-slate-400">${p2} 계약이 없습니다.</td></tr>`;

    document.getElementById('slide-1-risk').textContent = state.riskMessage;
    document.getElementById('slide-1-old-total').textContent = man(s.oldProjectedTotal);
    document.getElementById('slide-1-new-total').textContent = man(s.newTotal);
    document.getElementById('slide-1-old-sub').innerHTML = `월 ${won(s.oldMonthly)} 기준 (100세납 적용) | 완납 나이: <strong>${Math.floor(s.oldPaidAge)}세</strong>`;
    document.getElementById('slide-1-new-sub').innerHTML = `월 ${won(s.newMonthly)} 기준 (100세납 적용) | 완납 나이: <strong>${Math.floor(s.newPaidAge)}세</strong>`;

    const s2_leg_old = document.getElementById('slide-2-legend-old'); if(s2_leg_old) s2_leg_old.innerHTML = `<span class="w-5 h-5 bg-slate-300 rounded-full"></span> ${p1} 유지 (갱신형 100세납 포함)`;
    const s2_leg_new = document.getElementById('slide-2-legend-new'); if(s2_leg_new) s2_leg_new.innerHTML = `<span class="w-5 h-5 bg-blue-600 rounded-full"></span> ${p2} 적용`;
    
    renderChart(s, p1, p2);
    document.getElementById('slide-2-summary-cost').innerHTML = s.save >= 0 ? `총 지출 예상액 <strong class="text-blue-600 text-3xl ml-2">${man(s.save)} 절감</strong> 가능` : `보장 강화로 지출액 <strong class="text-rose-600 text-3xl ml-2">${man(Math.abs(s.save))} 증가</strong>`;
    document.getElementById('slide-2-summary-age').innerHTML = `${p1} ${Math.floor(s.oldPaidAge)}세까지의 납입 기간을 <strong class="text-slate-900 text-3xl ml-2">${Math.floor(s.newPaidAge)}세 완납</strong> 구조로 변경`;

    const buildBenefitItem = (title, content, isNew = false) => {
        const highlightColor = isNew ? 'text-blue-600' : 'text-slate-800';
        let bodyHtml = '';

        if (typeof content === 'string') {
            const lines = content.split('\n').filter(l => l.trim());
            bodyHtml = lines.map(line => {
                if (line.includes('최초 1회') || line.includes('매년 보상')) {
                    return `<p class="font-black text-slate-800 text-[18px] mt-2 mb-0.5">${escape(line)}</p>`;
                } else {
                    const escapedLine = escape(line);
                    const highlighted = escapedLine.replace(/\[(.*?)\]/g, `<strong class="${highlightColor}">[$1]</strong>`);
                    return `<li class="flex items-start gap-2.5 text-[17px] text-slate-600 leading-snug"><span class="mt-2 w-2 h-2 rounded-full bg-slate-300 shrink-0"></span><span class="flex-1">${highlighted}</span></li>`;
                }
            }).join('');
        } else {
            let catKey = '';
            if (title === '암 보장 전문성') catKey = 'Cancer';
            else if (title === '뇌질환 보장 범위') catKey = 'Brain';
            else if (title === '심장질환 보장 범위') catKey = 'Heart';

            const tpl = BENEFIT_TEMPLATES[catKey];
            bodyHtml = tpl.map(item => {
                if (item.type === 'header') {
                    return `<p class="font-black text-slate-800 text-[18px] mt-2 mb-0.5">${escape(item.text)}</p>`;
                } else if (item.type === 'custom') {
                    const val = content[item.key];
                    if (!val) return ''; 
                    const highlighted = escape(val).replace(/\[(.*?)\]/g, `<strong class="${highlightColor}">[$1]</strong>`);
                    return `<li class="flex items-start gap-2.5 text-[17px] text-slate-600 leading-snug"><span class="mt-2 w-2 h-2 rounded-full bg-slate-300 shrink-0"></span><span class="flex-1">${highlighted}</span></li>`;
                } else {
                    const val = content[item.key] || '0만원';
                    const highlighted = `<strong class="${highlightColor}">[${escape(val)}]</strong>`;
                    return `<li class="flex items-start gap-2.5 text-[17px] text-slate-600 leading-snug"><span class="mt-2 w-2 h-2 rounded-full bg-slate-300 shrink-0"></span><span class="flex-1">${escape(item.label)} ${highlighted}</span></li>`;
                }
            }).join('');
        }
        
        return `<div>
            <p class="text-sm font-black text-slate-400 uppercase mb-1">${title}</p>
            <ul class="space-y-1">${bodyHtml}</ul>
        </div>`;
    };
        
    const categories = ['Cancer', 'Brain', 'Heart', 'Surgery', 'Other'], catNames = ['암 보장 전문성', '뇌질환 보장 범위', '심장질환 보장 범위', '수술 보장', '기타 필수 보장'];
    document.getElementById('slide-3-old-benefits').innerHTML = categories.map((c, i) => buildBenefitItem(catNames[i], state.benefits['old'+c], false)).join('');
    document.getElementById('slide-3-new-benefits').innerHTML = categories.map((c, i) => buildBenefitItem(catNames[i], state.benefits['new'+c], true)).join('');
        
    document.getElementById('slide-3-comment-left').textContent = state.comment1;
    document.getElementById('slide-3-comment-right').textContent = `월 보험료 ${won(s.newMonthly)} 내에서 생애 전반의 리스크를 커버하도록 구성되었습니다.`;
    document.getElementById('slide-3-old-title').textContent = oldActive.map(p=>p.company).filter((v,i,a)=>a.indexOf(v)===i).join(', ') || `${p1} 보험`;
    document.getElementById('slide-3-new-title').textContent = newActive.map(p=>p.company).filter((v,i,a)=>a.indexOf(v)===i).join(', ') || `${p2} 보험`;

    const s4_col_old = document.getElementById('slide-4-col-old'); if(s4_col_old) s4_col_old.textContent = `${p1} 보험`;
    const s4_col_new = document.getElementById('slide-4-col-new'); if(s4_col_new) s4_col_new.textContent = `${p2} 보험`;

    function parseManwon(str) {
        if (!str) return 0;
        let s = String(str).replace(/,/g, '').trim();
        let val = 0;
        if (s.includes('억')) {
            const parts = s.split('억');
            val += Number(parts[0].replace(/[^\d.]/g, '') || 1) * 10000;
            if (parts[1]) val += Number(parts[1].replace(/[^\d.]/g, '') || 0);
        } else {
            val += Number(s.replace(/[^\d.]/g, '') || 0);
        }
        return val;
    }

    function formatManwonToKorean(val) {
        if (val === 0) return '0원';
        if (val < 10000) return val.toLocaleString() + '만원';
        const eok = Math.floor(val / 10000);
        const rest = val % 10000;
        return rest > 0 ? `${eok}억 ${rest.toLocaleString()}만원` : `${eok}억원`;
    }

    const renderSimLines = (data, lines, isNew) => {
        const color = isNew ? 'text-blue-700' : 'text-slate-700';
        const muted = isNew ? 'text-blue-400' : 'text-slate-400';
        return lines.map(l => {
            const vStr = String(data[l.key] || '').trim();
            const parsed = parseManwon(vStr);
            if(parsed === 0) return `<div class="text-[17px] mb-2 flex justify-between ${muted}"><span class="font-bold">${l.label}</span> <span>0원 (보장없음)</span></div>`;
            
            const totalManwon = parsed * l.mult;
            const totalStr = l.mult > 1 ? `<span class="text-sm font-normal text-${isNew?'blue-600':'slate-500'} ml-2">(${formatManwonToKorean(totalManwon)})</span>` : '';
            
            return `<div class="text-[17px] mb-2 flex justify-between items-end border-b border-dashed ${isNew?'border-blue-200/50':'border-slate-200'} pb-1"><span class="${isNew?'text-blue-800/70':'text-slate-500'} font-bold">${l.label}</span> <span class="text-right"><strong class="${color} font-black text-xl">${escape(vStr)}</strong>${l.mult > 1 ? ` <span class="text-sm font-bold text-slate-500">× ${l.mult}회</span>` : ''}${totalStr}</span></div>`;
        }).join('');
    };

    const oldC = state.benefits.oldCancer;
    const newC = state.benefits.newCancer;

    if (document.getElementById('slide-4-old-y1')) {
        const linesY1 = [
            {label: '암진단비', key: 'cancerDiag', mult: 1}, 
            {label: '암치료진단비', key: 'cancerTreatDiag', mult: 1},
            {label: '비급여 수술', key: 'cancerNonSurg', mult: 1}, 
            {label: '비급여 항암', key: 'cancerNonAnti', mult: 1}
        ];
        document.getElementById('slide-4-old-y1').innerHTML = renderSimLines(oldC, linesY1, false);
        document.getElementById('slide-4-new-y1').innerHTML = renderSimLines(newC, linesY1, true);

        const linesY2 = [
            {label: '암치료진단비', key: 'cancerTreatDiag', mult: 1},
            {label: '비급여 항암 (3회 치료)', key: 'cancerNonAnti', mult: 1}, 
            {label: '비급여 방사선', key: 'cancerNonRad', mult: 1}
        ];
        document.getElementById('slide-4-old-y2').innerHTML = renderSimLines(oldC, linesY2, false);
        document.getElementById('slide-4-new-y2').innerHTML = renderSimLines(newC, linesY2, true);

        const linesY3 = [
            {label: '암치료진단비', key: 'cancerTreatDiag', mult: 1},
            {label: '비급여 항암 (2회 치료)', key: 'cancerNonAnti', mult: 1}
        ];
        document.getElementById('slide-4-old-y3').innerHTML = renderSimLines(oldC, linesY3, false);
        document.getElementById('slide-4-new-y3').innerHTML = renderSimLines(newC, linesY3, true);

        const calcSimTotal = (data) => parseManwon(data.cancerDiag) + (parseManwon(data.cancerTreatDiag) * 3) + parseManwon(data.cancerNonSurg) + (parseManwon(data.cancerNonAnti) * 3) + parseManwon(data.cancerNonRad);
        
        document.getElementById('slide-4-old-total').textContent = formatManwonToKorean(calcSimTotal(oldC));
        document.getElementById('slide-4-new-total').textContent = formatManwonToKorean(calcSimTotal(newC));
    }

    setTimeout(adjustScale, 0);
}

function renderChart(s, p1 = '추천1', p2 = '추천2') {
    const svg = document.getElementById('svg-chart');
    const margin = { t: 150, r: 80, b: 80, l: 110 }, w = 1000, h = 550;
    const maxAge = 100, maxVal = Math.max(s.oldProjectedTotal, s.newTotal, 1) * 1.25;
    const getX = age => margin.l + (age - s.age) / (maxAge - s.age) * (w - margin.l - margin.r);
    const getY = val => (h - margin.b) - (val / maxVal) * (h - margin.b - margin.t);

    let html = `
        <defs>
            <linearGradient id="area-old" x1="0" y1="0" x2="0" y2="1">
                <stop offset="0%" stop-color="#94a3b8" stop-opacity="0.25"/>
                <stop offset="100%" stop-color="#f8fafc" stop-opacity="0.0"/>
            </linearGradient>
            <linearGradient id="area-new" x1="0" y1="0" x2="0" y2="1">
                <stop offset="0%" stop-color="#3b82f6" stop-opacity="0.35"/>
                <stop offset="100%" stop-color="#eff6ff" stop-opacity="0.0"/>
            </linearGradient>
        </defs>
        <line x1="${margin.l}" y1="${h-margin.b}" x2="${w-margin.r}" y2="${h-margin.b}" stroke="#cbd5e1" stroke-width="2.5" />
        <text x="${margin.l}" y="${h-margin.b+35}" text-anchor="middle" class="text-[18px] fill-slate-500 font-bold">${s.age}세</text>
        <text x="${w-margin.r}" y="${h-margin.b+35}" text-anchor="middle" class="text-[18px] fill-slate-500 font-bold">100세</text>
    `;
        
    [maxVal * 0.5, maxVal].forEach(v => {
        const y = getY(v);
        html += `<line x1="${margin.l}" y1="${y}" x2="${w-margin.r}" y2="${y}" stroke="#f1f5f9" stroke-width="2" /><text x="${margin.l-15}" y="${y+6}" text-anchor="end" class="text-[16px] fill-slate-400 font-bold">${man(v)}</text>`;
    });

    const oldActivePlans = state.oldPlans.filter(p => p.active);
    const newActivePlans = state.newPlans.filter(p => p.active);

    const getPathD = (plans) => {
        if (!plans.length) return `M ${getX(s.age)} ${getY(0)} L ${getX(100)} ${getY(0)}`;
        const ages = new Set();
        for(let a = s.age; a <= 100; a++) ages.add(a); 
        plans.forEach(p => ages.add(s.age + getActualMonths(p, s.age) / 12)); 
        const sortedAges = Array.from(ages).sort((a,b)=>a-b);
                
        let d = `M ${getX(sortedAges[0])} ${getY(0)}`;
        for (let i = 1; i < sortedAges.length; i++) {
            const age = sortedAges[i];
            const sum = plans.reduce((acc, p) => acc + calcCumulativeAtAge(p, s.age, age), 0);
            d += ` L ${getX(age)} ${getY(sum)}`;
        }
        return d;
    };

    const oldPathD = getPathD(oldActivePlans);
    const newPathD = getPathD(newActivePlans);

    const getAreaD = (d) => d + ` L ${getX(100)} ${getY(0)} L ${getX(s.age)} ${getY(0)} Z`;

    html += `<path d="${getAreaD(oldPathD)}" fill="url(#area-old)" />`;
    html += `<path d="${oldPathD}" fill="none" stroke="#94a3b8" stroke-width="6" class="chart-path-old" />`;
        
    html += `<path d="${getAreaD(newPathD)}" fill="url(#area-new)" />`;
    html += `<path d="${newPathD}" fill="none" stroke="#2563eb" stroke-width="8" />`;

    let oldLabelOffsets = {};
    oldActivePlans.forEach(p => {
        if (!p.company) return;
        const pAge = s.age + (getActualMonths(p, s.age) / 12);
        const px = getX(pAge);
        const cumulativeSum = oldActivePlans.reduce((sum, plan) => sum + calcCumulativeAtAge(plan, s.age, pAge), 0);
        const py = getY(cumulativeSum);
                
        const ageKey = Math.floor(pAge);
        oldLabelOffsets[ageKey] = (oldLabelOffsets[ageKey] || 0) + 1;
        const yOffset = -25 - (oldLabelOffsets[ageKey] - 1) * 45; 

        html += `<circle cx="${px}" cy="${py}" r="8" fill="#94a3b8" />
            <g transform="translate(${px}, ${py + yOffset})">
                <rect x="-70" y="-35" width="140" height="34" rx="8" fill="white" stroke="#94a3b8" stroke-width="2" />
                <text text-anchor="middle" y="-12" class="text-[16px] font-bold" fill="#334155">${escape(p.company)} ${Math.floor(pAge)}세</text>
            </g>`;
    });

    let newLabelOffsets = {};
    newActivePlans.forEach(p => {
        if (!p.company) return;
        const pAge = s.age + (getActualMonths(p, s.age) / 12);
        const px = getX(pAge);
        const cumulativeSum = newActivePlans.reduce((sum, plan) => sum + calcCumulativeAtAge(plan, s.age, pAge), 0);
        const py = getY(cumulativeSum);
                
        const ageKey = Math.floor(pAge);
        newLabelOffsets[ageKey] = (newLabelOffsets[ageKey] || 0) + 1;
        const yOffset = -28 - (newLabelOffsets[ageKey] - 1) * 48;

        html += `<circle cx="${px}" cy="${py}" r="10" fill="#2563eb" />
            <g transform="translate(${px}, ${py + yOffset})">
                <rect x="-75" y="-38" width="150" height="36" rx="10" fill="#2563eb" />
                <text text-anchor="middle" y="-13" class="text-[17px] font-bold" fill="white">${escape(p.company)} ${Math.floor(pAge)}세</text>
            </g>`;
    });

    const badgeY = 60;
    const oldBadgeX = margin.l;
    html += `
        <g transform="translate(${oldBadgeX}, ${badgeY})">
            <rect x="0" y="-50" width="240" height="90" rx="16" fill="#f8fafc" stroke="#cbd5e1" stroke-width="3" />
            <text x="120" y="-15" text-anchor="middle" class="text-[18px] font-bold" fill="#64748b">${p1} 유지 예상 총액</text>
            <text x="120" y="25" text-anchor="middle" class="text-[32px] font-black" fill="#1e293b">${man(s.oldProjectedTotal)}</text>
        </g>
    `;

    const newBadgeX = w - margin.r - 240;
    html += `
        <g transform="translate(${newBadgeX}, ${badgeY})">
            <rect x="0" y="-50" width="240" height="90" rx="16" fill="#2563eb" />
            <text x="120" y="-15" text-anchor="middle" class="text-[18px] font-bold" fill="#dbeafe">${p2} 리모델링 총액</text>
            <text x="120" y="25" text-anchor="middle" class="text-[34px] font-black" fill="white">${man(s.newTotal)}</text>
            <circle cx="240" cy="-50" r="24" fill="#facc15" stroke="white" stroke-width="4" />
            <text x="240" y="-43" text-anchor="middle" class="text-[14px] font-black" fill="#854d0e">BEST</text>
        </g>
    `;

    const fullX = getX(100);
    html += `<circle cx="${fullX}" cy="${getY(s.oldProjectedTotal)}" r="8" fill="#cbd5e1" />`;
    html += `<circle cx="${fullX}" cy="${getY(s.newTotal)}" r="10" fill="white" stroke="#2563eb" stroke-width="5" />`;

    svg.innerHTML = html;
}

function resetData() { 
    if (confirm('데이터를 초기화하시겠습니까?')) { 
        state = JSON.parse(JSON.stringify(INITIAL_DATA)); 
        initForm(); 
        showToast("초기화되었습니다."); 
    } 
}

function resetOldPremiumsToZero() {
    if (!confirm(`${state.planName1 || '추천1'} 계약의 월 보험료만 모두 0원으로 초기화하시겠습니까?`)) return;
    state.oldPlans.forEach(p => { p.monthly = 0; });
    renderInputLists();
    updateSlidesAndMetrics();
    showToast(`${state.planName1 || '추천1'} 보험료가 0원으로 초기화되었습니다.`);
}

function resetBenefitsToZero() {
    if (!ensureManagerSelected()) return;
    const p1 = state.planName1 || '추천1';
    const p2 = state.planName2 || '추천2';
    if (!confirm(`입력된 모든 보장 금액(${p1}/${p2})을 0원으로 초기화하시겠습니까?`)) return;

    const resetStruct = (obj) => {
        for(let key in obj) {
            if (key.includes('Custom')) obj[key] = '';
            else obj[key] = '0만원';
        }
    };
    
    resetStruct(state.benefits.oldCancer);
    resetStruct(state.benefits.oldBrain);
    resetStruct(state.benefits.oldHeart);
    resetStruct(state.benefits.newCancer);
    resetStruct(state.benefits.newBrain);
    resetStruct(state.benefits.newHeart);
    
    state.benefits.oldSurgery = '질병수술비 [ 0만원 ]\n상해수술비 [ 0만원 ]';
    state.benefits.oldOther = '가족일상생활배상책임 [ 0만원 ]\n운전자/상해 특약 [ 0만원 ]';
    state.benefits.newSurgery = '질병 1-5종 수술비 [ 0만원 ]\n상해 1-5종 수술비 [ 0만원 ]';
    state.benefits.newOther = '가족일상생활배상책임 [ 0만원 ]\n운전자/상해 보장 강화 [ 0만원 ]';

    ['oldSurgery', 'newSurgery', 'oldOther', 'newOther'].forEach(key => {
        const el = document.getElementById('benefit-' + key);
        if(el) el.value = state.benefits[key];
    });

    renderBenefitForm();
    updateSlidesAndMetrics();
    showToast(`${p1} 및 ${p2} 보장 금액이 0원으로 초기화되었습니다.`);
}

function applyRenewalToAll() { 
    state.oldPlans.forEach(p => { 
        p.maturity = '100세'; 
        p.remainingCount = (100 - Number(state.customerAge)) * 12; 
        p.renewalCycle = p.renewalCycle || 3; 
        p.increaseRate = p.increaseRate || 10; 
    }); 
    renderInputLists(); 
    updateSlidesAndMetrics(); 
    showToast("100세납 조건이 적용되었습니다."); 
}

function copyJson() { 
    navigator.clipboard.writeText(JSON.stringify(state, null, 2)); 
    showToast("JSON 데이터가 복사되었습니다."); 
}

async function waitForCaptureReady() {
    try {
        if (document.fonts && document.fonts.ready) await document.fonts.ready;
    } catch (e) {}
    await new Promise(resolve => requestAnimationFrame(() => requestAnimationFrame(resolve)));
}

function buildCaptureClone(ids, wrapperMode = 'single') {
    const stage = document.createElement('div');
    stage.style.position = 'fixed';
    stage.style.left = '-99999px';
    stage.style.top = '0';
    stage.style.zIndex = '-1';
    stage.style.pointerEvents = 'none';
    stage.style.background = '#e2e8f0';
    stage.style.padding = wrapperMode === 'merged' ? '40px' : '0';
    stage.style.width = wrapperMode === 'merged' ? '1280px' : '1200px';
    stage.style.boxSizing = 'border-box';

    const wrapper = document.createElement('div');
    wrapper.style.width = '1200px';
    wrapper.style.display = 'flex';
    wrapper.style.flexDirection = 'column';
    wrapper.style.gap = wrapperMode === 'merged' ? '60px' : '0';
    wrapper.style.background = wrapperMode === 'merged' ? '#e2e8f0' : 'transparent';

    ids.forEach(id => {
        const original = document.getElementById(id);
        const clone = original.cloneNode(true);
        clone.style.transform = 'none';
                
        clone.style.width = '1200px';
        clone.style.minHeight = '1200px';
        clone.style.height = 'auto';
        clone.style.margin = '0';
        clone.style.boxShadow = 'none';
        clone.style.borderRadius = '16px'; 
        clone.style.overflow = 'hidden';
        clone.querySelectorAll('.no-print, button').forEach(el => el.remove());
        wrapper.appendChild(clone);
    });

    stage.appendChild(wrapper);
    document.body.appendChild(stage);
    return { stage, wrapper };
}

async function renderCanvasFromIds(ids, mode = 'single') {
    await waitForCaptureReady();
    const { stage, wrapper } = buildCaptureClone(ids, mode);
    try {
        const canvas = await html2canvas(wrapper, {
            scale: 2,
            useCORS: true,
            backgroundColor: mode === 'merged' ? '#e2e8f0' : '#ffffff',
            windowWidth: wrapper.scrollWidth,
            windowHeight: wrapper.scrollHeight,
            width: wrapper.scrollWidth,
            height: wrapper.scrollHeight,
            scrollX: 0,
            scrollY: 0
        });
        return canvas;
    } finally {
        stage.remove();
    }
}

async function saveSlide(id, name) {
    try {
        showToast('이미지 저장 중입니다.');
        const canvas = await renderCanvasFromIds([id], 'single');
        const link = document.createElement('a');
        link.download = `${name}_${state.customerName}.png`;
        link.href = canvas.toDataURL('image/png');
        link.click();
        showToast('이미지가 저장되었습니다.');
    } catch (e) {
        console.error(e);
        showToast('저장 실패');
    }
}

async function saveAllSlides() {
    try {
        showToast('결과 4장을 합친 이미지를 생성 중입니다.');
        const canvas = await renderCanvasFromIds(['slide-1', 'slide-2', 'slide-3', 'slide-4'], 'merged');
        const link = document.createElement('a');
        link.download = `보험비교리포트_4장통합_${state.customerName}.png`;
        link.href = canvas.toDataURL('image/png');
        link.click();
        showToast('통합 이미지가 저장되었습니다.');
    } catch (e) {
        console.error(e);
        showToast('통합 저장 실패');
    }
}

function exportFullHtml() { 
    if (!(state.managerName === '선택안함' && state.customerName === '홍길동' && Number(state.customerAge) === 0)) {
        return; 
    }
    const source = document.documentElement.outerHTML; 
    const stateRegex = /let state = JSON\.parse\(JSON\.stringify\(INITIAL_DATA\)\);/; 
    const injected = source.replace(stateRegex, `let state = ${JSON.stringify(state, null, 2)};`); 
    const blob = new Blob([injected], { type: 'text/html' }); 
    const url = URL.createObjectURL(blob); 
    const link = document.createElement('a'); 
    link.href = url; 
    link.download = `보험비교리포트_${state.customerName}.html`; 
    link.click(); 
}

window.onload = initForm;
</script>
</body>
</html>
