<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2028 대입 개편: 5등급 → 9등급 정밀 환산기</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700&display=swap');
        body { font-family: 'Noto Sans KR', sans-serif; }
    </style>
</head>
<body class="bg-gray-50 text-gray-800">

    <div class="max-w-2xl mx-auto min-h-screen flex flex-col shadow-lg bg-white">
        
        <header class="bg-indigo-600 text-white p-6 rounded-b-2xl shadow-md">
            <h1 class="text-2xl font-bold mb-2"><i class="fas fa-calculator mr-2"></i>5등급 → 9등급 정밀 환산기</h1>
            <p class="text-indigo-100 text-sm">부산시교육청 실증 데이터 기반 (보간법 적용)</p>
        </header>

        <main class="flex-grow p-6">
            
            <div class="bg-indigo-50 border-l-4 border-indigo-500 p-4 mb-6 rounded-r">
                <h3 class="font-bold text-indigo-700 mb-1">업데이트 사항</h3>
                <p class="text-sm text-gray-600">
                    단순 변환 방식 대신 <strong>정밀 보간법</strong>을 적용했습니다.<br>
                    예: 5등급제 평균 <strong>2.00</strong> → 9등급제 <strong>3.44</strong> (정확)
                </p>
            </div>

            <div class="space-y-4" id="subject-list">
                </div>

            <button onclick="addSubject()" class="w-full mt-4 py-3 border-2 border-dashed border-gray-300 rounded-lg text-gray-500 hover:border-indigo-500 hover:text-indigo-600 transition flex items-center justify-center font-medium">
                <i class="fas fa-plus mr-2"></i> 과목 추가하기
            </button>

        </main>

        <footer class="bg-gray-900 text-white p-6 rounded-t-2xl mt-4 shadow-[0_-5px_15px_rgba(0,0,0,0.1)]">
            <div class="flex justify-between items-center mb-4">
                <div class="text-gray-400 text-sm">총 이수단위</div>
                <div class="font-bold text-lg" id="total-credits">0</div>
            </div>
            <div class="h-px bg-gray-700 mb-4"></div>
            <div class="flex justify-between items-end mb-2">
                <div class="text-gray-300">5등급제 평균</div>
                <div class="text-2xl font-bold" id="avg-5">0.00</div>
            </div>
            <div class="flex justify-between items-end">
                <div class="text-yellow-400 font-bold">9등급 환산 평균</div>
                <div class="text-4xl font-bold text-yellow-400" id="avg-9">0.00</div>
            </div>
            <button onclick="resetAll()" class="w-full mt-6 bg-gray-700 hover:bg-gray-600 text-gray-300 py-2 rounded text-sm transition">
                초기화
            </button>
        </footer>

    </div>

    <script>
        // ---------------------------------------------------------
        // [핵심 데이터] 부산광역시교육청 및 경기진협 분석 기준표
        // gpa5: 5등급제 점수, gpa9: 9등급제 환산 점수
        // ---------------------------------------------------------
        const referenceTable = [
            { gpa5: 1.00, gpa9: 1.64 }, // 1학기 실제 분포 기준 (보수적 적용)
            { gpa5: 1.16, gpa9: 1.81 },
            { gpa5: 1.33, gpa9: 2.18 },
            { gpa5: 1.50, gpa9: 2.48 },
            { gpa5: 1.66, gpa9: 2.76 },
            { gpa5: 1.83, gpa9: 3.07 },
            { gpa5: 2.00, gpa9: 3.44 }, // 사용자가 지적한 2.0 -> 3.44 구간
            { gpa5: 2.50, gpa9: 4.22 },
            { gpa5: 3.00, gpa9: 5.08 },
            { gpa5: 4.00, gpa9: 6.69 },
            { gpa5: 5.00, gpa9: 9.00 }
        ];

        let subjects = [
            { id: 1, name: '국어', credit: 4, grade: 1 },
            { id: 2, name: '수학', credit: 4, grade: 1 },
            { id: 3, name: '영어', credit: 4, grade: 2 },
            { id: 4, name: '통합사회', credit: 3, grade: 2 },
            { id: 5, name: '통합과학', credit: 3, grade: 2 }
        ];

        // 5등급제 평균 점수를 받아 9등급제로 환산(보간법)하는 함수
        function convertTo9(gpa5) {
            if (gpa5 < 1.0) return 1.0;
            if (gpa5 > 5.0) return 9.0;

            for (let i = 0; i < referenceTable.length - 1; i++) {
                const curr = referenceTable[i];
                const next = referenceTable[i+1];

                if (gpa5 >= curr.gpa5 && gpa5 <= next.gpa5) {
                    // 선형 보간 공식 적용
                    // Y = y1 + (x - x1) * (y2 - y1) / (x2 - x1)
                    const ratio = (gpa5 - curr.gpa5) / (next.gpa5 - curr.gpa5);
                    const result = curr.gpa9 + ratio * (next.gpa9 - curr.gpa9);
                    return result;
                }
            }
            return 9.00;
        }

        function renderSubjects() {
            const container = document.getElementById('subject-list');
            container.innerHTML = '';

            subjects.forEach((sub, index) => {
                const div = document.createElement('div');
                div.className = 'flex items-center gap-2 bg-white p-3 rounded-lg border shadow-sm';
                div.innerHTML = `
                    <div class="flex-grow">
                        <input type="text" placeholder="과목명" value="${sub.name}" 
                            onchange="updateSubject(${index}, 'name', this.value)"
                            class="w-full font-medium text-gray-700 bg-transparent focus:outline-none placeholder-gray-400">
                    </div>
                    <div class="flex items-center gap-2">
                        <div class="flex flex-col items-center">
                            <label class="text-[10px] text-gray-400">단위</label>
                            <input type="number" value="${sub.credit}" min="1" max="10"
                                onchange="updateSubject(${index}, 'credit', this.value)"
                                class="w-12 text-center p-1 bg-gray-50 border rounded text-sm font-bold text-gray-600 focus:ring-1 focus:ring-indigo-500 outline-none">
                        </div>
                        <div class="flex flex-col items-center">
                            <label class="text-[10px] text-indigo-500 font-bold">등급</label>
                            <select onchange="updateSubject(${index}, 'grade', this.value)"
                                class="w-14 text-center p-1 bg-indigo-50 border border-indigo-200 rounded text-sm font-bold text-indigo-700 focus:ring-1 focus:ring-indigo-500 outline-none">
                                <option value="1" ${sub.grade == 1? 'selected' : ''}>1</option>
                                <option value="2" ${sub.grade == 2? 'selected' : ''}>2</option>
                                <option value="3" ${sub.grade == 3? 'selected' : ''}>3</option>
                                <option value="4" ${sub.grade == 4? 'selected' : ''}>4</option>
                                <option value="5" ${sub.grade == 5? 'selected' : ''}>5</option>
                            </select>
                        </div>
                        <button onclick="removeSubject(${index})" class="text-gray-300 hover:text-red-500 ml-1">
                            <i class="fas fa-times-circle"></i>
                        </button>
                    </div>
                `;
                container.appendChild(div);
            });
            calculate();
        }

        function addSubject() {
            subjects.push({ id: Date.now(), name: '', credit: 3, grade: 2 });
            renderSubjects();
        }

        function removeSubject(index) {
            if (subjects.length > 1) {
                subjects.splice(index, 1);
                renderSubjects();
            } else {
                alert("최소 한 개의 과목은 있어야 합니다.");
            }
        }

        function updateSubject(index, key, value) {
            if (key === 'credit' |

| key === 'grade') {
                value = Number(value);
            }
            subjects[index][key] = value;
            calculate();
        }

        function calculate() {
            let totalCredits = 0;
            let weightedSum5 = 0;

            // 1. 5등급제 가중 평균 먼저 계산
            subjects.forEach(sub => {
                const credit = parseInt(sub.credit) |

| 0;
                const grade5 = parseInt(sub.grade) |

| 5;

                totalCredits += credit;
                weightedSum5 += (grade5 * credit);
            });

            const avg5 = totalCredits === 0? 0 : (weightedSum5 / totalCredits);

            // 2. 계산된 5등급 평균을 9등급으로 환산 (보간법 적용)
            const avg9 = totalCredits === 0? 0 : convertTo9(avg5);

            document.getElementById('total-credits').innerText = totalCredits;
            document.getElementById('avg-5').innerText = avg5.toFixed(2);
            document.getElementById('avg-9').innerText = avg9.toFixed(2);
        }

        function resetAll() {
            if(confirm('모든 내용을 초기화하시겠습니까?')) {
                subjects = [{ id: 1, name: '국어', credit: 4, grade: 1 }];
                renderSubjects();
            }
        }

        renderSubjects();
    </script>
</body>
</html>
