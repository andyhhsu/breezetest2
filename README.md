<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>三星產品考題產生器 (v4.1 - 更新機型列表)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .question-card {
            background-color: #f9fafb; /* 淡灰色背景 */
            border: 1px solid #e5e7eb; /* 灰色邊框 */
            border-radius: 0.5rem; /* 圓角 */
            padding: 1rem; /* 內邊距 */
            margin-bottom: 1rem; /* 下邊距 */
            box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06); /* 細微陰影 */
        }
        .answer-input {
            margin-top: 0.5rem;
            width: 100%;
            padding: 0.5rem;
            border: 1px solid #d1d5db;
            border-radius: 0.375rem;
        }
        .result-feedback {
            margin-top: 0.5rem;
            padding: 0.5rem;
            border-radius: 0.375rem;
        }
        .correct-answer {
            background-color: #dcfce7; /* 綠色提示背景 */
            border-left: 4px solid #22c55e; /* 綠色左邊框 */
        }
        .incorrect-answer {
            background-color: #fee2e2; /* 紅色提示背景 */
            border-left: 4px solid #ef4444; /* 紅色左邊框 */
        }
        .btn {
            padding: 0.5rem 1rem;
            border-radius: 0.375rem;
            font-weight: 600;
            transition: background-color 0.3s ease;
            cursor: pointer;
        }
        .btn-primary {
            background-color: #3b82f6; /* 藍色按鈕 */
            color: white;
        }
        .btn-primary:hover {
            background-color: #2563eb; /* 深藍色按鈕 (滑鼠懸停) */
        }
        /* 響應式設計 */
        @media (max-width: 640px) {
            .container {
                padding: 1rem; /* 手機視圖時調整內邊距 */
            }
            .input-group {
                flex-direction: column;
                gap: 0.5rem;
            }
            .input-group > * {
                width: 100%;
            }
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center py-8">
    <div class="container mx-auto max-w-5xl bg-white p-6 sm:p-8 rounded-lg shadow-xl"> {/* 頁面寬度調整為 max-w-5xl */}
        <header class="text-center mb-6">
            <h1 class="text-3xl font-bold text-blue-600">三星產品考題產生器</h1>
            <p class="text-gray-600 mt-2">選擇產品機型與規格類別，從(模擬的)網路資料產生考題。</p>
        </header>

        <div class="input-group space-y-4 sm:space-y-0 sm:flex sm:space-x-4 mb-6">
            <div class="flex-grow">
                <label for="modelNameSelect" class="block text-sm font-medium text-gray-700 mb-1">產品機型:</label>
                <select id="modelNameSelect" name="modelNameSelect"
                        class="mt-1 block w-full px-3 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                    </select>
            </div>
            <div class="flex-grow">
                <label for="dataSourceSelect" class="block text-sm font-medium text-gray-700 mb-1">選擇規格類別:</label>
                <select id="dataSourceSelect" name="dataSourceSelect"
                        class="mt-1 block w-full px-3 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm">
                    </select>
            </div>
        </div>

        <button id="generateButton" class="btn btn-primary w-full sm:w-auto mb-6">
            產生考題
        </button>

        <div id="quizArea" class="mt-8">
            {/* 考題將顯示於此 */}
        </div>

        <div id="submitArea" class="mt-6 hidden">
             <button id="submitButton" class="btn btn-primary w-full sm:w-auto">送出答案</button>
        </div>

        <div id="scoreArea" class="mt-8">
            {/* 分數將顯示於此 */}
        </div>

         <div id="noResultsMessage" class="mt-6 text-center text-gray-500 hidden">
            <p>找不到符合條件的題目，請嘗試其他機型或資料類別。</p>
        </div>
    </div>

    <script>
        // --- 預定義產品機型列表 (依照使用者最新指定) ---
        const productModels = [
            // S Series
            "Galaxy S25 Ultra", "Galaxy S25+", "Galaxy S25", "Galaxy S25 Edge",
            "Galaxy S24 Ultra", "Galaxy S24+", "Galaxy S24",
            // Z Series
            "Galaxy Z Fold6", "Galaxy Z Flip6",
            // A Series (Future Models)
            "Galaxy A56", "Galaxy A36", "Galaxy A26",
            // Tab S Series (Future Models)
            "Galaxy Tab S10 Ultra", "Galaxy Tab S10+", "Galaxy Tab S10", "Galaxy Tab S10 FE",
            // Watch Series (Future Models)
            "Galaxy Watch7", "Galaxy Watch Ultra",
            // Buds Series (Future Models)
            "Galaxy Buds3 Pro", "Galaxy Buds3"
            // 舊機型已移除
        ];

        // --- 模擬從網路搜尋到的產品規格資料 ---
        // !!! 注意：未來機型的規格與價格主要基於市場傳聞與預測，僅供參考 !!!
        // 此處保留先前已填充的資料，未列於上方 productModels 的機型將無法透過下拉選單選到
        const fetchedProductSpecs = {
            // S25 Series (預期/傳聞規格與價格)
            "galaxy s25 ultra": {
                "處理器型號": "Snapdragon 8 Gen 4 for Galaxy (預期)", "處理器核心數": "8核心 (預期)", "RAM": "12GB/16GB (預期)", "儲存空間": "256GB/512GB/1TB (預期)",
                "主螢幕尺寸": "6.9吋 (預期)", "主螢幕解析度": "QHD+ (預期)", "主螢幕技術": "Dynamic AMOLED 3X (預期)", "主螢幕最高更新頻率": "1-144Hz (預期)",
                "主相機最高畫素": "2億畫素 (改良感光元件預期)", "望遠鏡頭1畫素及光學變焦": "5000萬畫素 (5倍光學變焦，預期)", "望遠鏡頭2畫素及光學變焦": "5000萬畫素 (可變焦鏡頭傳聞)", "超廣角鏡頭畫素": "1200萬畫素 (預期)", "前鏡頭畫素": "1200萬畫素 (預期)", "錄影最高解析度": "8K (預期更高幀率)",
                "電池容量": "5000mAh (或更高，預期)", "有線快速充電瓦數": "支援45W (或更高，預期)", "無線充電": "支援 (可能提升瓦數)",
                "防水防塵等級": "IP68 (預期)", "內建 S Pen": "是 (預期)", "NFC功能": "支援 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$43,900 起 (高度預期，可能更高)"
            },
            "galaxy s25+": {
                "處理器型號": "Snapdragon 8 Gen 4 / Exynos 2500 (預期)", "RAM": "12GB (預期)", "儲存空間": "256GB/512GB (預期)",
                "主螢幕尺寸": "6.7吋 (預期)", "主螢幕技術": "Dynamic AMOLED 3X (預期)", "主螢幕最高更新頻率": "120Hz (預期)",
                "主相機最高畫素": "5000萬畫素 (預期)", "望遠鏡頭1畫素及光學變焦": "1000萬畫素 (3倍光學變焦，預期)", "超廣角鏡頭畫素": "1200萬畫素 (預期)",
                "電池容量": "4900mAh (或更高，預期)", "有線快速充電瓦數": "支援45W (預期)",
                "防水防塵等級": "IP68 (預期)", "NFC功能": "支援 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$34,900 起 (高度預期)"
            },
            "galaxy s25": {
                "處理器型號": "Snapdragon 8 Gen 4 / Exynos 2500 (預期)", "RAM": "8GB/12GB (預期)", "儲存空間": "128GB/256GB (預期)",
                "主螢幕尺寸": "6.2吋 (預期)", "主螢幕技術": "Dynamic AMOLED 3X (預期)", "主螢幕最高更新頻率": "120Hz (預期)",
                "主相機最高畫素": "5000萬畫素 (預期)", "望遠鏡頭1畫素及光學變焦": "1000萬畫素 (3倍光學變焦，預期)", "超廣角鏡頭畫素": "1200萬畫素 (預期)",
                "電池容量": "4000mAh (或更高，預期)", "有線快速充電瓦數": "支援25W (預期)",
                "防水防塵等級": "IP68 (預期)", "NFC功能": "支援 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$27,900 起 (高度預期)"
            },
            "galaxy s25 edge": {
                "處理器型號": "Snapdragon 8 Gen 4 for Galaxy (傳聞)", "RAM": "12GB (傳聞)", "儲存空間": "256GB/512GB (傳聞)",
                "主螢幕尺寸": "6.7吋 (傳聞，可能為曲面)", "主螢幕技術": "Dynamic AMOLED (傳聞)", "主螢幕最高更新頻率": "120Hz (傳聞)",
                "主相機最高畫素": "2億畫素主鏡頭 (傳聞)", "超廣角鏡頭畫素": "1200萬畫素 (傳聞)",
                "電池容量": "3900mAh (傳聞，相對較小)", "有線快速充電瓦數": "支援25W (傳聞)",
                "防水防塵等級": "IP68 (預期)", "NFC功能": "支援 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "約 NT$37,900 起 (傳聞，基於歐元價格推算)"
            },
            "galaxy s24 ultra": {
                "處理器型號": "Qualcomm Snapdragon 8 Gen 3 for Galaxy", "處理器核心數": "8核心", "RAM": "12GB",
                "主螢幕尺寸": "6.8吋", "主螢幕解析度": "3120x1440 pixels (QHD+)", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "2億畫素", "望遠鏡頭1畫素及光學變焦": "5000萬畫素 (5倍光學變焦)", "望遠鏡頭2畫素及光學變焦": "1000萬畫素 (3倍光學變焦)", "超廣角鏡頭畫素": "1200萬畫素", "前鏡頭畫素": "1200萬畫素", "錄影最高解析度": "8K (30fps)",
                "電池容量": "5000mAh", "有線快速充電瓦數": "支援45W", "無線充電": "支援",
                "防水防塵等級": "IP68", "內建 S Pen": "是", "NFC功能": "支援", "作業系統": "Android 14",
                "建議售價": "NT$43,900 起 (12G/256G 上市價)"
            },
            "galaxy s24+": {
                "處理器型號": "Snapdragon 8 Gen 3 for Galaxy / Exynos 2400", "處理器核心數": "8核心 (Snapdragon) / 10核心 (Exynos)", "RAM": "12GB",
                "主螢幕尺寸": "6.7吋", "主螢幕解析度": "3120x1440 pixels (QHD+)", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "5000萬畫素", "望遠鏡頭1畫素及光學變焦": "1000萬畫素 (3倍光學變焦)", "超廣角鏡頭畫素": "1200萬畫素", "前鏡頭畫素": "1200萬畫素", "錄影最高解析度": "8K (30fps)",
                "電池容量": "4900mAh", "有線快速充電瓦數": "支援45W", "無線充電": "支援",
                "防水防塵等級": "IP68", "NFC功能": "支援", "作業系統": "Android 14",
                "建議售價": "NT$34,900 起 (12G/256G 上市價)"
            },
            "galaxy s24": {
                "處理器型號": "Snapdragon 8 Gen 3 for Galaxy / Exynos 2400", "處理器核心數": "8核心 (Snapdragon) / 10核心 (Exynos)", "RAM": "8GB",
                "主螢幕尺寸": "6.2吋", "主螢幕解析度": "2340x1080 pixels (FHD+)", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "5000萬畫素", "望遠鏡頭1畫素及光學變焦": "1000萬畫素 (3倍光學變焦)", "超廣角鏡頭畫素": "1200萬畫素", "前鏡頭畫素": "1200萬畫素", "錄影最高解析度": "8K (30fps)",
                "電池容量": "4000mAh", "有線快速充電瓦數": "支援25W", "無線充電": "支援",
                "防水防塵等級": "IP68", "NFC功能": "支援", "作業系統": "Android 14",
                "建議售價": "NT$27,900 起 (8G/256G 上市價)"
            },
            "galaxy z fold6": {
                "處理器型號": "Snapdragon 8 Gen 3 for Galaxy (預期)", "處理器核心數": "8核心", "RAM": "12GB",
                "內頁螢幕尺寸": "7.6吋 (預期)", "封面螢幕尺寸": "6.3吋 (預期)", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "5000萬畫素", "望遠鏡頭1畫素及光學變焦": "1000萬畫素 (3倍光學變焦)", "超廣角鏡頭畫素": "1200萬畫素",
                "前鏡頭畫素": "400萬畫素 (螢幕下 - 內頁)", "封面螢幕前鏡頭畫素": "1000萬畫素",
                "電池容量": "4400mAh (預期)", "有線快速充電瓦數": "支援25W", "無線充電": "支援",
                "防水防塵等級": "IPX8 (預期)", "內建 S Pen": "支援 (可能需另購)", "NFC功能": "支援", "作業系統": "Android 14 (或更新)",
                "建議售價": "NT$59,888 起 (12G/256G 預期/上市價)"
            },
            "galaxy z flip6": {
                "處理器型號": "Snapdragon 8 Gen 3 for Galaxy (預期)", "處理器核心數": "8核心", "RAM": "8GB/12GB",
                "主螢幕尺寸": "6.7吋", "封面螢幕尺寸": "3.4吋 (或更大傳聞)", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "5000萬畫素 (預期升級)", "超廣角鏡頭畫素": "1200萬畫素", "前鏡頭畫素": "1000萬畫素",
                "電池容量": "4000mAh (預期提升)", "有線快速充電瓦數": "支援25W", "無線充電": "支援",
                "防水防塵等級": "IPX8 (預期，或傳聞IP48)", "NFC功能": "支援", "作業系統": "Android 14 (或更新)",
                "建議售價": "NT$35,888 起 (12G/256G 預期/上市價)"
            },
            "galaxy a56": {
                "處理器型號": "Exynos 1580 (預期)", "RAM": "8GB/12GB (預期)", "儲存空間": "256GB (預期)",
                "主螢幕尺寸": "6.7吋 (預期)", "主螢幕技術": "Super AMOLED (預期)", "主螢幕最高更新頻率": "120Hz (預期)",
                "主相機最高畫素": "5000萬畫素 (預期)", "超廣角鏡頭畫素": "1200萬畫素 (預期)", "微距鏡頭畫素": "500萬畫素 (預期)",
                "電池容量": "5000mAh (預期)", "有線快速充電瓦數": "支援45W (預期提升)",
                "防水防塵等級": "IP67 (預期)", "NFC功能": "支援 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$16,990 起 (8G/256G 預期)"
            },
            "galaxy a36": {
                "處理器型號": "Snapdragon 6 Gen 3 / Exynos (預期)", "RAM": "8GB (預期)", "儲存空間": "128GB/256GB (預期)",
                "主螢幕尺寸": "6.7吋 (預期)", "主螢幕技術": "Super AMOLED (預期)", "主螢幕最高更新頻率": "120Hz (預期)",
                "主相機最高畫素": "5000萬畫素 (預期)", "超廣角鏡頭畫素": "800萬畫素 (預期)", "微距鏡頭畫素": "500萬畫素 (預期)",
                "電池容量": "5000mAh (預期)", "有線快速充電瓦數": "支援45W (預期提升)",
                "防水防塵等級": "IP67 (預期)", "NFC功能": "支援 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$13,990 起 (8G/128G 預期)"
            },
            "galaxy a26": {
                "處理器型號": "Exynos 1380 (或同級，預期)", "RAM": "6GB/8GB (預期)", "儲存空間": "128GB (預期)",
                "主螢幕尺寸": "6.7吋 (預期)", "主螢幕技術": "Super AMOLED (預期)", "主螢幕最高更新頻率": "120Hz (預期)",
                "主相機最高畫素": "5000萬畫素 (預期)", "超廣角鏡頭畫素": "800萬畫素 (預期)", "微距鏡頭畫素": "200萬畫素 (預期)",
                "電池容量": "5000mAh (預期)", "有線快速充電瓦數": "支援25W (預期)",
                "防水防塵等級": "IP67 (預期)", "NFC功能": "支援 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$11,990 起 (8G/128G 預期)"
            },
            "galaxy tab s10 ultra": {
                "處理器型號": "Snapdragon 8 Gen 4 for Galaxy / Dimensity 9300+ (傳聞)", "RAM": "12GB/16GB (預期)", "儲存空間": "256GB/512GB/1TB (預期)",
                "主螢幕尺寸": "14.6吋 (預期)", "主螢幕技術": "Dynamic AMOLED 2X (預期)", "主螢幕最高更新頻率": "120Hz (預期)",
                "主相機最高畫素": "1300萬畫素 (預期)", "超廣角鏡頭畫素": "800萬畫素 (預期)", "前鏡頭畫素":"1200萬畫素 + 1200萬畫素超廣角 (預期)",
                "電池容量": "11200mAh (預期)", "有線快速充電瓦數": "支援45W (預期)",
                "防水防塵等級": "IP68 (預期)", "內建 S Pen": "是 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$39,990 起 (Wi-Fi, 12G/256G 預期)"
            },
            "galaxy tab s10+": {
                "處理器型號": "Snapdragon 8 Gen 4 for Galaxy / Dimensity (預期)", "RAM": "12GB (預期)", "儲存空間": "256GB/512GB (預期)",
                "主螢幕尺寸": "12.4吋 (預期)", "主螢幕技術": "Dynamic AMOLED 2X (預期)", "主螢幕最高更新頻率": "120Hz (預期)",
                "電池容量": "10090mAh (預期)", "有線快速充電瓦數": "支援45W (預期)",
                "防水防塵等級": "IP68 (預期)", "內建 S Pen": "是 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$32,990 起 (Wi-Fi, 12G/256G 預期)"
            },
            "galaxy tab s10": {
                "處理器型號": "Snapdragon 8 Gen 3 for Galaxy / Dimensity (預期)", "RAM": "8GB/12GB (預期)", "儲存空間": "128GB/256GB (預期)",
                "主螢幕尺寸": "11吋 (預期)", "主螢幕技術": "Dynamic AMOLED 2X (預期)", "主螢幕最高更新頻率": "120Hz (預期)",
                "電池容量": "8400mAh (預期)", "有線快速充電瓦數": "支援45W (預期)",
                "防水防塵等級": "IP68 (預期)", "內建 S Pen": "是 (預期)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$23,990 起 (Wi-Fi, 8G/128G 預期)"
            },
            "galaxy tab s10 fe": {
                "處理器型號": "Exynos 1580 (傳聞)", "RAM": "8GB/12GB (傳聞)", "儲存空間": "128GB/256GB (傳聞)",
                "主螢幕尺寸": "10.9吋 (FE) / 13.1吋 (FE+ 傳聞)", "主螢幕技術": "LCD (傳聞)", "主螢幕最高更新頻率": "90Hz (傳聞)",
                "電池容量": "8000mAh (FE) / 10090mAh (FE+ 傳聞)", "有線快速充電瓦數": "支援45W (傳聞)",
                "防水防塵等級": "IP68 (傳聞)", "內建 S Pen": "是 (傳聞)", "作業系統": "Android 15 (預期)",
                "建議售價": "NT$15,900 起 (Wi-Fi, 8G/128G 預期)"
            },
            "galaxy watch7": {
                "處理器型號": "Exynos W1000 (或更新3nm晶片，預期)", "RAM": "2GB (預期)", "儲存空間": "32GB (預期)",
                "主螢幕尺寸": "1.3吋(40mm) / 1.5吋(44mm) (預期)", "主螢幕技術": "Super AMOLED (預期)",
                "電池容量": "300mAh(40mm) / 425mAh(44mm) (預期)", "無線充電": "支援 (預期)",
                "防水防塵等級": "5ATM + IP68 (預期)", "NFC功能": "支援 (預期)", "作業系統": "Wear OS (預期更新版本)",
                "健康感測器": "心率, ECG, 血氧, 睡眠, 體溫, 可能有血糖監測傳聞", "錶殼材質": "鋁合金 (預期)",
                "建議售價": "NT$9,990 起 (40mm GPS版預期)"
            },
            "galaxy watch ultra": {
                "處理器型號": "Exynos W1000 (或更新3nm晶片，預期)", "RAM": "2GB (或更高，預期)", "儲存空間": "32GB (或更高，預期)",
                "主螢幕尺寸": "1.5吋 (或更大，47mm傳聞)", "主螢幕技術": "Super AMOLED (高亮度預期)", "錶殼材質": "鈦金屬 (傳聞)",
                "電池容量": "590mAh (傳聞，較大容量)", "無線充電": "支援 (預期)",
                "防水防塵等級": "10ATM + IP68 + MIL-STD-810H (預期更強固)", "NFC功能": "支援 (預期)", "作業系統": "Wear OS (預期更新版本)",
                "健康感測器": "心率, ECG, 血氧, 睡眠, 體溫, 可能有血糖監測傳聞, 雙頻GPS",
                "建議售價": "NT$21,990 起 (預期，定位高階)"
            },
            "galaxy buds3 pro": {
                "藍牙版本": "5.4 (預期)", "主動式降噪": "支援 (更強效果預期)", "單耳續航(ANC開)": "約5-6小時 (預期)", "總續航(ANC開)": "約18-22小時 (預期)",
                "防水防塵等級": "IPX7 (或更高，預期)", "特色功能": "空間音訊, Hi-Fi音質 (24-bit), AI通話降噪 (預期)", "作業系統": "N/A",
                "建議售價": "NT$7,490 (預期)"
            },
            "galaxy buds3": {
                "藍牙版本": "5.4 (預期)", "主動式降噪": "支援 (預期)", "單耳續航(ANC開)": "約6小時 (預期)", "總續航(ANC開)": "約24小時 (預期)",
                "防水防塵等級": "IP57 (預期)", "特色功能": "全新設計 (豆芽式傳聞), AI通話降噪 (預期)", "作業系統": "N/A",
                "建議售價": "NT$4,990 (預期)"
            },
            // --- 以下為先前已部分填充的機型，此處主要補充價格或確認核心規格 ---
            "galaxy s23 ultra": {
                "處理器型號": "Qualcomm Snapdragon 8 Gen 2 for Galaxy", "處理器核心數": "8核心", "RAM": "8GB/12GB",
                "主螢幕尺寸": "6.8吋", "主螢幕解析度": "3088x1440 pixels (QHD+)", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "2億畫素", "望遠鏡頭1畫素及光學變焦": "1000萬畫素 (3倍光學變焦)", "望遠鏡頭2畫素及光學變焦": "1000萬畫素 (10倍光學變焦)", "超廣角鏡頭畫素": "1200萬畫素", "前鏡頭畫素": "1200萬畫素", "錄影最高解析度": "8K (30fps)",
                "電池容量": "5000mAh", "有線快速充電瓦數": "支援45W", "無線充電": "支援",
                "防水防塵等級": "IP68", "內建 S Pen": "是", "NFC功能": "支援", "作業系統": "Android 13 (可升級)",
                "建議售價": "NT$40,900 起 (12G/256G 上市價)"
            },
            "galaxy s23 fe": {
                "處理器型號": "Snapdragon 8 Gen 1 / Exynos 2200", "處理器核心數": "8核心", "RAM": "8GB",
                "主螢幕尺寸": "6.4吋", "主螢幕解析度": "2340x1080 pixels (FHD+)", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "5000萬畫素", "望遠鏡頭1畫素及光學變焦": "800萬畫素 (3倍光學變焦)", "超廣角鏡頭畫素": "1200萬畫素", "前鏡頭畫素": "1000萬畫素", "錄影最高解析度": "8K (24fps)",
                "電池容量": "4500mAh", "有線快速充電瓦數": "支援25W", "無線充電": "支援",
                "防水防塵等級": "IP68", "NFC功能": "支援", "作業系統": "Android 13 (可升級)",
                "建議售價": "NT$19,490 起 (8G/128G 上市價)"
            },
            "galaxy z fold5": {
                "處理器型號": "Snapdragon 8 Gen 2 for Galaxy", "處理器核心數": "8核心", "RAM": "12GB",
                "內頁螢幕尺寸": "7.6吋", "封面螢幕尺寸": "6.2吋", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "5000萬畫素", "望遠鏡頭1畫素及光學變焦": "1000萬畫素 (3倍光學變焦)", "超廣角鏡頭畫素": "1200萬畫素",
                "前鏡頭畫素": "400萬畫素 (螢幕下)", "封面螢幕前鏡頭畫素": "1000萬畫素",
                "電池容量": "4400mAh", "有線快速充電瓦數": "支援25W", "無線充電": "支援",
                "防水防塵等級": "IPX8", "內建 S Pen": "支援 (需另購特定版本)", "NFC功能": "支援", "作業系統": "Android 13 (可升級)",
                "建議售價": "NT$56,888 起 (12G/256G 上市價)"
            },
            "galaxy z flip5": {
                "處理器型號": "Snapdragon 8 Gen 2 for Galaxy", "處理器核心數": "8核心", "RAM": "8GB",
                "主螢幕尺寸": "6.7吋", "封面螢幕尺寸": "3.4吋", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "1200萬畫素", "超廣角鏡頭畫素": "1200萬畫素", "前鏡頭畫素": "1000萬畫素",
                "電池容量": "3700mAh", "有線快速充電瓦數": "支援25W", "無線充電": "支援",
                "防水防塵等級": "IPX8", "NFC功能": "支援", "作業系統": "Android 13 (可升級)",
                "建議售價": "NT$32,888 起 (8G/256G 上市價)"
            },
            "galaxy a55 5g": {
                "處理器型號": "Exynos 1480", "處理器核心數": "8核心", "RAM": "8GB",
                "主螢幕尺寸": "6.6吋", "主螢幕解析度": "1080x2340 pixels (FHD+)", "主螢幕技術": "Super AMOLED", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "5000萬畫素 (OIS)", "超廣角鏡頭畫素": "1200萬畫素", "微距鏡頭畫素": "500萬畫素", "前鏡頭畫素": "3200萬畫素", "錄影最高解析度": "4K (30fps)",
                "電池容量": "5000mAh", "有線快速充電瓦數": "支援25W", "無線充電": "不支援",
                "防水防塵等級": "IP67", "NFC功能": "支援", "作業系統": "Android 14",
                "建議售價": "NT$14,990 起 (8G/128G 上市價)"
            },
            "galaxy a35 5g": {
                "處理器型號": "Exynos 1380", "處理器核心數": "8核心", "RAM": "6GB/8GB",
                "主螢幕尺寸": "6.6吋", "主螢幕解析度": "1080x2340 pixels (FHD+)", "主螢幕技術": "Super AMOLED", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "5000萬畫素 (OIS)", "超廣角鏡頭畫素": "800萬畫素", "微距鏡頭畫素": "500萬畫素", "前鏡頭畫素": "1300萬畫素", "錄影最高解析度": "4K (30fps)",
                "電池容量": "5000mAh", "有線快速充電瓦數": "支援25W", "無線充電": "不支援",
                "防水防塵等級": "IP67", "NFC功能": "支援", "作業系統": "Android 14",
                "建議售價": "NT$11,990 起 (6G/128G 上市價)"
            },
            "galaxy tab s9 ultra": {
                "處理器型號": "Qualcomm Snapdragon 8 Gen 2 for Galaxy", "處理器核心數": "8核心", "RAM": "12GB/16GB",
                "主螢幕尺寸": "14.6吋", "主螢幕解析度": "2960x1848 pixels", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "1300萬畫素", "超廣角鏡頭畫素": "800萬畫素", "前鏡頭畫素":"1200萬畫素 + 1200萬畫素超廣角", "錄影最高解析度": "4K (30/60fps)",
                "電池容量": "11200mAh", "有線快速充電瓦數": "支援45W", "無線充電": "不支援",
                "防水防塵等級": "IP68", "內建 S Pen": "是 (隨附)", "NFC功能": "不支援", "作業系統": "Android 13 (可升級)",
                "建議售價": "NT$39,990 起 (Wi-Fi, 12G/256G 上市價)"
            },
            "galaxy tab s9": {
                "處理器型號": "Qualcomm Snapdragon 8 Gen 2 for Galaxy", "處理器核心數": "8核心", "RAM": "8GB/12GB",
                "主螢幕尺寸": "11吋", "主螢幕解析度": "2560x1600 pixels (WQXGA)", "主螢幕技術": "Dynamic AMOLED 2X", "主螢幕最高更新頻率": "120Hz",
                "主相機最高畫素": "1300萬畫素", "前鏡頭畫素": "1200萬畫素 (超廣角)", "錄影最高解析度": "4K (30fps)",
                "電池容量": "8400mAh", "有線快速充電瓦數": "支援45W", "無線充電": "不支援",
                "防水防塵等級": "IP68", "內建 S Pen": "是 (隨附)", "NFC功能": "不支援", "作業系統": "Android 13 (可升級)",
                "建議售價": "NT$23,990 起 (Wi-Fi, 8G/128G 上市價)"
            },
            "galaxy watch6": {
                "處理器型號": "Exynos W930 (雙核心 1.4GHz)", "RAM": "2GB", "儲存空間": "16GB",
                "主螢幕尺寸": "1.5吋 (44mm) / 1.3吋 (40mm)", "主螢幕解析度": "480x480 pixels (44mm) / 432x432 pixels (40mm)", "主螢幕技術": "Super AMOLED",
                "電池容量": "425mAh (44mm) / 300mAh (40mm)", "無線充電": "支援 (WPC無線充電)",
                "防水防塵等級": "5ATM + IP68", "NFC功能": "支援", "作業系統": "Wear OS Powered by Samsung",
                "健康感測器": "心率偵測, ECG, 血氧飽和度, 睡眠追蹤, BIA身體組成分析, 體溫感測", "錶殼材質": "鋁合金",
                "建議售價": "NT$9,990 起 (40mm BT版上市價)"
            },
             "galaxy watch6 classic": { // Watch6 Classic 的資料
                "處理器型號": "Exynos W930 (雙核心 1.4GHz)", "RAM": "2GB", "儲存空間": "16GB",
                "主螢幕尺寸": "1.5吋 (47mm) / 1.3吋 (43mm)", "主螢幕解析度": "480x480 pixels (47mm) / 432x432 pixels (43mm)", "主螢幕技術": "Super AMOLED", "特色功能": "實體旋轉錶圈",
                "電池容量": "425mAh (47mm) / 300mAh (43mm)", "無線充電": "支援 (WPC無線充電)",
                "防水防塵等級": "5ATM + IP68", "NFC功能": "支援", "作業系統": "Wear OS Powered by Samsung",
                "健康感測器": "心率偵測, ECG, 血氧飽和度, 睡眠追蹤, BIA身體組成分析, 體溫感測", "錶殼材質": "不鏽鋼",
                "建議售價": "NT$12,990 起 (43mm BT版上市價)"
            },
            "galaxy buds fe": {
                "藍牙版本": "5.2", "主動式降噪": "支援", "單耳續航(ANC開)": "約6小時", "總續航(ANC開)": "約21小時",
                "防水防塵等級": "IPX2 (僅耳機)", "特色功能": "舒適耳翼設計", "作業系統": "N/A",
                "建議售價": "NT$2,990 (上市價)"
            },
            "galaxy buds2 pro": {
                "藍牙版本": "5.3", "主動式降噪": "智慧主動式降噪", "單耳續航(ANC開)": "約5小時", "總續航(ANC開)": "約18小時",
                "防水防塵等級": "IPX7", "特色功能": "Hi-Fi高保真音效 (24bit), 360度環繞音訊", "作業系統": "N/A",
                "建議售價": "NT$7,490 (上市價)"
            }
        };

        // --- 定義規格類別及相關的規格項目 ---
        const dataSources = {
            "processor": {
                name: "處理器與記憶體",
                specItems: ["處理器型號", "處理器核心數", "RAM", "儲存空間"],
                questionTemplates: [
                    (model, specName) => `請問 ${model} 的 ${specName} 是 _______。`,
                    (model, specName) => `${model} 所搭載的 ${specName} 為何？ _______。`
                ]
            },
            "display": {
                name: "螢幕規格",
                specItems: ["主螢幕尺寸", "主螢幕解析度", "主螢幕技術", "主螢幕最高更新頻率", "內頁螢幕尺寸", "封面螢幕尺寸"],
                questionTemplates: [
                    (model, specName) => `${model} 的 ${specName} 是 _______。`,
                    (model, specName) => `關於 ${model} 的 ${specName}，其規格為 _______。`
                ]
            },
            "camera": {
                name: "相機規格",
                specItems: ["主相機最高畫素", "望遠鏡頭1畫素及光學變焦", "望遠鏡頭2畫素及光學變焦", "超廣角鏡頭畫素", "微距鏡頭畫素", "前鏡頭畫素", "封面螢幕前鏡頭畫素", "前鏡頭超廣角畫素", "錄影最高解析度"],
                questionTemplates: [
                    (model, specName) => `${model} 的 ${specName} 為 _______。`,
                    (model, specName) => `請問 ${model} 在 ${specName} 方面的規格是 _______？`
                ]
            },
            "battery": {
                name: "電池與充電",
                specItems: ["電池容量", "有線快速充電瓦數", "無線充電", "單耳續航(ANC開)", "總續航(ANC開)"],
                questionTemplates: [
                    (model, specName) => `請問 ${model} 的 ${specName} 是 _______。`,
                    (model, specName) => `${model} 是否支援 ${specName}？ (請回答支援瓦數/是/否/支援與否/小時數) _______。`
                ]
            },
            "other_features": {
                name: "其他特色/規格",
                specItems: ["防水防塵等級", "內建 S Pen", "NFC功能", "作業系統", "健康感測器", "藍牙版本", "特色功能", "錶殼材質"],
                questionTemplates: [
                    (model, specName) => `${model} 的 ${specName} 為何？ (請回答等級/是/否/支援與否/版本/材質/詳細內容) _______。`,
                    (model, specName) => `請問 ${model} 是否具備 ${specName} 功能/規格？ (請回答是/否/支援與否/版本/材質/詳細內容) _______。`
                ]
            },
            "price_info": {
                name: "價格資訊",
                specItems: ["建議售價"],
                questionTemplates: [
                    (model, specName) => `請問 ${model} 的 ${specName} 大約是多少？ _______。`,
                    (model, specName) => `${model} 的 ${specName} 為何？ _______。`
                ]
            }
        };

        // --- DOM 元素 ---
        const modelNameSelect = document.getElementById('modelNameSelect');
        const dataSourceSelect = document.getElementById('dataSourceSelect');
        const generateButton = document.getElementById('generateButton');
        const quizArea = document.getElementById('quizArea');
        const submitArea = document.getElementById('submitArea');
        const submitButton = document.getElementById('submitButton');
        const scoreArea = document.getElementById('scoreArea');
        const noResultsMessage = document.getElementById('noResultsMessage');

        let currentQuizData = [];
        const MAX_QUESTIONS = 10;

        // --- 初始化 ---
        function initialize() {
            productModels.sort((a, b) => {
                const aIsFuture = ["S25", "A56", "A36", "A26", "Tab S10", "Watch7", "Watch Ultra", "Buds3"].some(p => a.includes(p));
                const bIsFuture = ["S25", "A56", "A36", "A26", "Tab S10", "Watch7", "Watch Ultra", "Buds3"].some(p => b.includes(p));
                const aIsS = a.includes("Galaxy S");
                const bIsS = b.includes("Galaxy S");
                const aIsZ = a.includes("Galaxy Z");
                const bIsZ = b.includes("Galaxy Z");

                if (aIsFuture && !bIsFuture) return -1; // 未來機型優先
                if (!aIsFuture && bIsFuture) return 1;

                if (aIsS && !bIsS) return -1; // S系列優先於非S/Z
                if (!aIsS && bIsS) return 1;
                if (aIsZ && !bIsZ) return -1; // Z系列優先於非S/Z
                if (!aIsZ && bIsZ) return 1;
                
                return a.localeCompare(b); // 其他按字母
            });

            productModels.forEach(model => {
                const option = document.createElement('option');
                option.value = model.toLowerCase().replace(/\s+/g, ' ').trim();
                option.textContent = model;
                modelNameSelect.appendChild(option);
            });
            
            const defaultModel = productModels.find(m => m === "Galaxy S25 Ultra") || productModels[0];
            if (defaultModel) {
                modelNameSelect.value = defaultModel.toLowerCase().replace(/\s+/g, ' ').trim();
            }


            for (const key in dataSources) {
                const option = document.createElement('option');
                option.value = key;
                option.textContent = dataSources[key].name;
                dataSourceSelect.appendChild(option);
            }

            generateButton.addEventListener('click', generateQuestionsHandlerWrapper); // 使用包裝器
            generateButton.setAttribute('data-listener-set', 'true'); // 初始設定監聽器標記
            submitButton.addEventListener('click', handleSubmit);
        }
        
        function generateQuestionsHandlerWrapper() { // 包裝器函數
            generateQuestions();
        }

        // --- 考題產生邏輯 ---
        function generateQuestions() {
            const selectedModelKey = modelNameSelect.value;
            const selectedDataSourceKey = dataSourceSelect.value;
            const modelDisplayName = modelNameSelect.options[modelNameSelect.selectedIndex].text;

            const category = dataSources[selectedDataSourceKey];
            if (!category) {
                alert('無效的規格類別！');
                return;
            }

            const specsForModel = fetchedProductSpecs[selectedModelKey];

            quizArea.innerHTML = '';
            scoreArea.innerHTML = '';
            currentQuizData = [];
            noResultsMessage.classList.add('hidden');

            if (!specsForModel) {
                noResultsMessage.innerHTML = `<p>抱歉，目前「${modelDisplayName}」的詳細規格資料尚未完整載入於此範例中。</p><p>（特別是預期/未來機型，資料可能仍在收集中或此處未包含）</p>`;
                noResultsMessage.classList.remove('hidden');
                submitArea.classList.add('hidden');
                return;
            }

            let availableQuestions = [];
            for (const specItemName of category.specItems) {
                if (specsForModel.hasOwnProperty(specItemName) && specsForModel[specItemName] !== undefined && specsForModel[specItemName] !== null && String(specsForModel[specItemName]).trim() !== "") {
                    const specValue = specsForModel[specItemName];
                    const templateIndex = Math.floor(Math.random() * category.questionTemplates.length);
                    const questionText = category.questionTemplates[templateIndex](modelDisplayName, specItemName);
                    availableQuestions.push({ question: questionText, answer: specValue });
                }
            }

            if (availableQuestions.length === 0) {
                noResultsMessage.innerHTML = `<p>在「${modelDisplayName}」的「${category.name}」類別下，目前找不到可用的規格項目來產生題目。</p>`;
                noResultsMessage.classList.remove('hidden');
                submitArea.classList.add('hidden');
                return;
            }

            availableQuestions.sort(() => 0.5 - Math.random());
            const numQuestionsToGenerate = Math.min(Math.max(5, availableQuestions.length), MAX_QUESTIONS);
            currentQuizData = availableQuestions.slice(0, numQuestionsToGenerate);


            currentQuizData.forEach((qData, index) => {
                const questionId = `q-${selectedDataSourceKey}-${index}`;
                const questionCard = document.createElement('div');
                questionCard.className = 'question-card';

                const questionP = document.createElement('p');
                questionP.className = 'text-gray-800 mb-2';
                questionP.innerHTML = `<strong>題目 ${index + 1}:</strong> ${qData.question.replace('_______', `<span class="font-bold text-blue-600">(填空處)</span>`)}`;
                questionCard.appendChild(questionP);

                const answerInput = document.createElement('input');
                answerInput.type = 'text';
                answerInput.id = `ans-input-${questionId}`;
                answerInput.className = 'answer-input';
                answerInput.placeholder = '請在此輸入您的答案';
                questionCard.appendChild(answerInput);

                const feedbackDiv = document.createElement('div');
                feedbackDiv.id = `feedback-${questionId}`;
                feedbackDiv.className = 'result-feedback hidden';
                questionCard.appendChild(feedbackDiv);

                quizArea.appendChild(questionCard);
            });

            if (currentQuizData.length > 0) {
                submitArea.classList.remove('hidden');
                submitButton.disabled = false;
                generateButton.textContent = "產生考題"; // 重設按鈕文字
            } else {
                noResultsMessage.innerHTML = `<p>在「${modelDisplayName}」的「${category.name}」類別下，未能成功產生題目。</p>`;
                noResultsMessage.classList.remove('hidden');
                submitArea.classList.add('hidden');
            }
        }
        

        // --- 提交答案與計分邏輯 ---
        function handleSubmit() {
            let score = 0;
            const questionCards = quizArea.querySelectorAll('.question-card');

            if (questionCards.length !== currentQuizData.length) {
                scoreArea.innerHTML = `<p class="text-red-600 font-semibold">發生錯誤，無法計分。題目與答案數量不符。</p>`;
                return;
            }

            questionCards.forEach((card, index) => {
                const inputElement = card.querySelector('input[type="text"]');
                const feedbackElement = card.querySelector('.result-feedback');
                const userAnswer = inputElement.value.trim();
                const correctAnswer = currentQuizData[index].answer;

                feedbackElement.classList.remove('hidden', 'correct-answer', 'incorrect-answer');

                if (correctAnswer !== undefined && userAnswer.toLowerCase() === String(correctAnswer).toLowerCase()) {
                    score++;
                    feedbackElement.innerHTML = `<strong>您的答案:</strong> ${userAnswer || '(未作答)'} <br> <span class="text-green-600 font-semibold">✔ 正確</span>`;
                    feedbackElement.classList.add('correct-answer');
                } else {
                    feedbackElement.innerHTML = `<strong>您的答案:</strong> ${userAnswer || '(未作答)'} <br> <strong>正確答案:</strong> ${correctAnswer === undefined ? '未定義' : correctAnswer} <br> <span class="text-red-600 font-semibold">✘ 錯誤</span>`;
                    feedbackElement.classList.add('incorrect-answer');
                }
                inputElement.disabled = true;
            });

            scoreArea.innerHTML = `<h2 class="text-2xl font-bold mb-2">測驗結果</h2>
                                 <p class="text-xl">您答對了 ${score} / ${currentQuizData.length} 題。</p>`;
            submitButton.disabled = true;
            generateButton.textContent = "重新產生考題";
            
            // 移除舊的 generateButton 監聽器 (如果它是 handleSubmit 臨時添加的那個)
            const currentOnClick = generateButton.onclick;
            if (currentOnClick && currentOnClick.name === "newGenerateButtonHandler") { // 假設臨時處理函數叫這個名字
                 generateButton.removeEventListener('click', currentOnClick);
            }
            
            // 添加一個新的臨時監聽器，用於 "重新產生考題"
            const newGenerateButtonHandler = () => {
                submitButton.disabled = false;
                generateButton.textContent = "產生考題";
                quizArea.innerHTML = '';
                scoreArea.innerHTML = '';
                submitArea.classList.add('hidden');
                noResultsMessage.classList.add('hidden');
                generateButton.removeEventListener('click', newGenerateButtonHandler); // 移除自身
                
                // 重新綁定主要的 generateQuestions 監聽器
                if (!generateButton.hasAttribute('data-listener-set')) {
                     generateButton.addEventListener('click', generateQuestionsHandlerWrapper);
                     generateButton.setAttribute('data-listener-set', 'true');
                }
                generateQuestions(); // 呼叫產生新題目
            };
            // 移除主監聽器，添加臨時的
            generateButton.removeEventListener('click', generateQuestionsHandlerWrapper); 
            generateButton.removeAttribute('data-listener-set'); 
            generateButton.addEventListener('click', newGenerateButtonHandler);
        }
        document.addEventListener('DOMContentLoaded', initialize);
    </script>
</body>
</html>
