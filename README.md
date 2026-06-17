<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Finanças Red</title>
  
  <!-- Configurações PWA -->
  <link rel="manifest" href="manifest.json">
  <meta name="theme-color" content="#059669">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/2454/2454282.png">

  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- React & Babel CDN -->
  <script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body class="bg-slate-50 text-slate-800 font-sans antialiased">
  
  <div id="root"></div>

  <!-- Registro de Service Worker para PWA -->
  <script>
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('sw.js')
          .then(reg => console.log('PWA Service Worker Ativo!', reg.scope))
          .catch(err => console.log('Erro ao registrar Service Worker:', err));
      });
    }
  </script>

  <!-- Lógica do App Financeiro do Red -->
  <script type="text/babel">
    const { useState, useEffect, useRef } = React;

    const DEFAULT_CATEGORIES = [
      { id: 'moradia', name: 'Moradia', icon: '🏠', color: 'bg-blue-100 text-blue-700 border-blue-200' },
      { id: 'alimentacao', name: 'Alimentação', icon: '🍔', color: 'bg-amber-100 text-amber-700 border-amber-200' },
      { id: 'transporte', name: 'Transporte', icon: '🚗', color: 'bg-indigo-100 text-indigo-700 border-indigo-200' },
      { id: 'lazer', name: 'Lazer & Viagens', icon: '🎉', color: 'bg-purple-100 text-purple-700 border-purple-200' },
      { id: 'saude', name: 'Saúde', icon: '❤️', color: 'bg-rose-100 text-rose-700 border-rose-200' },
      { id: 'educacao', name: 'Educação', icon: '📚', color: 'bg-emerald-100 text-emerald-700 border-emerald-200' },
      { id: 'outros', name: 'Outros', icon: '🏷️', color: 'bg-slate-100 text-slate-700 border-slate-200' }
    ];

    function App() {
      // --- ESTADOS ---
      const [currentMonth, setCurrentMonth] = useState('2026-06');
      const [activeTab, setActiveTab] = useState('dashboard');
      const fileInputRef = useRef(null);
      
      const [monthlyBudgets, setMonthlyBudgets] = useState({
        '2026-06': { income: 5000, savingsGoal: 1000 }
      });

      const [expenses, setExpenses] = useState([
        { id: '1', description: 'Aluguel e Condomínio', amount: 1500, type: 'fixed', category: 'moradia', date: '2026-06-01' },
        { id: '2', description: 'Supermercado Mensal', amount: 650, type: 'fixed', category: 'alimentacao', date: '2026-06-02' },
        { id: '3', description: 'Combustível', amount: 300, type: 'one-time', category: 'transporte', date: '2026-06-05' },
        { id: '4', description: 'Jantar de Aniversário', amount: 180, type: 'one-time', category: 'lazer', date: '2026-06-08' },
        { id: '5', description: 'Farmácia', amount: 95, type: 'one-time', category: 'saude', date: '2026-06-10' },
        { id: '6', description: 'Curso Online de Tech', amount: 250, type: 'fixed', category: 'educacao', date: '2026-06-12' }
      ]);

      const defaultIncome = 5000;
      const defaultSavingsGoal = 1000;
      const activeBudget = monthlyBudgets[currentMonth] || { income: defaultIncome, savingsGoal: defaultSavingsGoal };
      const income = activeBudget.income;
      const savingsGoal = activeBudget.savingsGoal;

      const [newDesc, setNewDesc] = useState('');
      const [newAmount, setNewAmount] = useState('');
      const [newType, setNewType] = useState('fixed');
      const [newCategory, setNewCategory] = useState('moradia');
      const [newDate, setNewDate] = useState('2026-06-01');

      const [isEditingIncome, setIsEditingIncome] = useState(false);
      const [tempIncome, setTempIncome] = useState(income);
      const [isEditingGoal, setIsEditingGoal] = useState(false);
      const [tempGoal, setTempGoal] = useState(savingsGoal);

      const [searchTerm, setSearchTerm] = useState('');
      const [filterType, setFilterType] = useState('all');
      const [filterCategory, setFilterCategory] = useState('all');
      const [notification, setNotification] = useState(null);

      useEffect(() => {
        setTempIncome(income);
        setTempGoal(savingsGoal);
        setNewDate(`${currentMonth}-01`);
      }, [currentMonth, monthlyBudgets]);

      useEffect(() => {
        try {
          const savedBudgets = localStorage.getItem('red_finance_monthly_budgets_v2');
          const savedExpenses = localStorage.getItem('red_finance_expenses_v2');
          const savedCurrentMonth = localStorage.getItem('red_finance_current_month');

          if (savedBudgets) setMonthlyBudgets(JSON.parse(savedBudgets));
          if (savedExpenses) setExpenses(JSON.parse(savedExpenses));
          if (savedCurrentMonth) setCurrentMonth(savedCurrentMonth);
        } catch (e) {
          console.warn("LocalStorage inacessível.");
        }
      }, []);

      const saveData = (updatedBudgets, updatedExpenses, monthToSave = currentMonth) => {
        try {
          localStorage.setItem('red_finance_monthly_budgets_v2', JSON.stringify(updatedBudgets));
          localStorage.setItem('red_finance_expenses_v2', JSON.stringify(updatedExpenses));
          localStorage.setItem('red_finance_current_month', monthToSave);
        } catch (e) {}
      };

      const triggerNotification = (message, type = 'success') => {
        setNotification({ message, type });
        setTimeout(() => setNotification(null), 3000);
      };

      const formatMonthName = (monthStr) => {
        const [year, month] = monthStr.split('-');
        const months = [
          'Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho',
          'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'
        ];
        return `${months[parseInt(month, 10) - 1]} de ${year}`;
      };

      const navigateMonth = (direction) => {
        const [year, month] = currentMonth.split('-').map(Number);
        let newYear = year;
        let newMonth = month + direction;

        if (newMonth > 12) {
          newMonth = 1;
          newYear += 1;
        } else if (newMonth < 1) {
          newMonth = 12;
          newYear -= 1;
        }

        const newMonthStr = `${newYear}-${String(newMonth).padStart(2, '0')}`;
        setCurrentMonth(newMonthStr);
        saveData(monthlyBudgets, expenses, newMonthStr);
      };

      const handleMonthPickerChange = (e) => {
        if (e.target.value) {
          setCurrentMonth(e.target.value);
          saveData(monthlyBudgets, expenses, e.target.value);
        }
      };

      const handleSaveIncome = () => {
        const val = parseFloat(tempIncome);
        if (!isNaN(val) && val >= 0) {
          const updated = {
            ...monthlyBudgets,
            [currentMonth]: { ...activeBudget, income: val }
          };
          setMonthlyBudgets(updated);
          setIsEditingIncome(false);
          saveData(updated, expenses);
          triggerNotification("Renda atualizada para este mês!");
        }
      };

      const handleSaveGoal = () => {
        const val = parseFloat(tempGoal);
        if (!isNaN(val) && val >= 0) {
          const updated = {
            ...monthlyBudgets,
            [currentMonth]: { ...activeBudget, savingsGoal: val }
          };
          setMonthlyBudgets(updated);
          setIsEditingGoal(false);
          saveData(updated, expenses);
          triggerNotification("Meta de poupança atualizada!");
        }
      };

      const handleAddExpense = (e) => {
        e.preventDefault();
        const amountNum = parseFloat(newAmount);
        if (!newDesc.trim() || isNaN(amountNum) || amountNum <= 0) {
          triggerNotification("Informe descrição e valor válido.", "error");
          return;
        }

        const newExpense = {
          id: self.crypto ? crypto.randomUUID() : Math.random().toString(36).substring(2),
          description: newDesc.trim(),
          amount: amountNum,
          type: newType,
          category: newCategory,
          date: newDate || `${currentMonth}-01`
        };

        const updated = [newExpense, ...expenses];
        setExpenses(updated);
        saveData(monthlyBudgets, updated);

        setNewDesc('');
        setNewAmount('');
        triggerNotification("Gasto registrado!");
      };

      const handleDeleteExpense = (id) => {
        const updated = expenses.filter(exp => exp.id !== id);
        setExpenses(updated);
        saveData(monthlyBudgets, updated);
        triggerNotification("Item removido.", "info");
      };

      const handleSpreadsheetCellChange = (id, field, value) => {
        const updatedExpenses = expenses.map(exp => {
          if (exp.id === id) {
            let parsedValue = value;
            if (field === 'amount') parsedValue = parseFloat(value) || 0;
            return { ...exp, [field]: parsedValue };
          }
          return exp;
        });
        setExpenses(updatedExpenses);
        saveData(monthlyBudgets, updatedExpenses);
      };

      const handleImportPreviousFixed = () => {
        const [year, month] = currentMonth.split('-').map(Number);
        let prevYear = year;
        let prevMonth = month - 1;

        if (prevMonth < 1) {
          prevMonth = 12;
          prevYear -= 1;
        }

        const prevMonthStr = `${prevYear}-${String(prevMonth).padStart(2, '0')}`;
        const previousFixedExpenses = expenses.filter(
          exp => exp.date.substring(0, 7) === prevMonthStr && exp.type === 'fixed'
        );

        if (previousFixedExpenses.length === 0) {
          triggerNotification("Sem despesas fixas no mês anterior.", "error");
          return;
        }

        const importedExpenses = previousFixedExpenses.map(oldExp => {
          const day = oldExp.date.substring(8, 10);
          return {
            ...oldExp,
            id: self.crypto ? crypto.randomUUID() : Math.random().toString(36).substring(2),
            date: `${currentMonth}-${day}`
          };
        });

        const updated = [...importedExpenses, ...expenses];
        setExpenses(updated);
        saveData(monthlyBudgets, updated);
        triggerNotification(`${importedExpenses.length} despesas fixas copiadas!`);
      };

      const activeMonthExpenses = expenses.filter(
        exp => exp.date.substring(0, 7) === currentMonth
      );

      const totalFixed = activeMonthExpenses.filter(exp => exp.type === 'fixed').reduce((sum, exp) => sum + exp.amount, 0);
      const totalOneTime = activeMonthExpenses.filter(exp => exp.type === 'one-time').reduce((sum, exp) => sum + exp.amount, 0);
      const totalExpenses = totalFixed + totalOneTime;
      const remainingBalance = income - totalExpenses;
      const savingsGoalPercentage = Math.min(Math.round((remainingBalance / (savingsGoal || 1)) * 100), 100);

      const categoryTotals = DEFAULT_CATEGORIES.map(cat => {
        const total = activeMonthExpenses.filter(exp => exp.category === cat.id).reduce((sum, exp) => sum + exp.amount, 0);
        return { ...cat, total };
      }).filter(cat => cat.total > 0);

      const fixedPercentageOfIncome = income > 0 ? (totalFixed / income) * 100 : 0;
      const oneTimePercentageOfIncome = income > 0 ? (totalOneTime / income) * 100 : 0;
      const savingsPercentageOfIncome = income > 0 ? (remainingBalance / income) * 100 : 0;

      const filteredExpenses = activeMonthExpenses.filter(exp => {
        const matchesSearch = exp.description.toLowerCase().includes(searchTerm.toLowerCase());
        const matchesType = filterType === 'all' || exp.type === filterType;
        const matchesCategory = filterCategory === 'all' || exp.category === filterCategory;
        return matchesSearch && matchesType && matchesCategory;
      });

      const handleExportCSV = () => {
        if (activeMonthExpenses.length === 0) {
          triggerNotification("Sem gastos para exportar.", "error");
          return;
        }
        const headers = ['Data', 'Descricao', 'Categoria', 'Tipo', 'Valor (R$)'];
        const rows = activeMonthExpenses.map(exp => {
          const catName = DEFAULT_CATEGORIES.find(c => c.id === exp.category)?.name || 'Outros';
          return [exp.date, exp.description, catName, exp.type === 'fixed' ? 'Fixo' : 'Pontual', exp.amount.toFixed(2).replace('.', ',')];
        });
        const csvContent = "\uFEFF" + [headers.join(';'), ...rows.map(r => r.join(';'))].join('\n');
        const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.setAttribute("href", url);
        link.setAttribute("download", `financas_${currentMonth}.csv`);
        link.click();
        triggerNotification("Planilha exportada!");
      };

      const handleCopyToClipboardForSheets = () => {
        if (activeMonthExpenses.length === 0) {
          triggerNotification("Nenhum dado para copiar.", "error");
          return;
        }
        const headers = ['Data', 'Descrição', 'Categoria', 'Tipo', 'Valor (R$)'];
        const rows = activeMonthExpenses.map(exp => {
          const catName = DEFAULT_CATEGORIES.find(c => c.id === exp.category)?.name || 'Outros';
          return [exp.date, exp.description, catName, exp.type === 'fixed' ? 'Fixo' : 'Pontual', exp.amount.toString().replace('.', ',')];
        });
        const tsvContent = [headers.join('\t'), ...rows.map(r => r.join('\t'))].join('\n');
        const tempTextArea = document.createElement('textarea');
        tempTextArea.value = tsvContent;
        document.body.appendChild(tempTextArea);
        tempTextArea.select();
        document.execCommand('copy');
        document.body.removeChild(tempTextArea);
        triggerNotification("Copiado! Pode colar (Ctrl+V) no Excel/Sheets.");
      };

      const handleExportBackup = () => {
        const backupObj = { monthlyBudgets, expenses, backupDate: new Date().toISOString() };
        const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(backupObj, null, 2));
        const link = document.createElement("a");
        link.setAttribute("href", dataStr);
        link.setAttribute("download", `backup_financas_red.json`);
        link.click();
        triggerNotification("Backup baixado!");
      };

      const handleImportBackup = (e) => {
        const fileReader = new FileReader();
        const file = e.target.files[0];
        if (!file) return;

        fileReader.onload = (event) => {
          try {
            const parsed = JSON.parse(event.target.result);
            if (parsed.monthlyBudgets && parsed.expenses) {
              setMonthlyBudgets(parsed.monthlyBudgets);
              setExpenses(parsed.expenses);
              triggerNotification("Backup restaurado!");
            }
          } catch (err) {
            triggerNotification("Arquivo inválido.", "error");
          }
        };
        fileReader.readAsText(file);
      };

      return (
        <div className="min-h-screen bg-slate-50 text-slate-800 pb-12">
          {notification && (
            <div className={`fixed top-4 right-4 z-50 flex items-center gap-2 px-4 py-3 rounded-xl shadow-lg border transition-all duration-300 ${
              notification.type === 'error' ? 'bg-rose-50 border-rose-200 text-rose-800' :
              notification.type === 'info' ? 'bg-blue-50 border-blue-200 text-blue-800' :
              'bg-emerald-50 border-emerald-200 text-emerald-800'
            }`}>
              <span className="text-sm font-semibold">{notification.message}</span>
            </div>
          )}

          <header className="bg-gradient-to-r from-emerald-600 to-teal-700 text-white shadow-md">
            <div className="max-w-7xl mx-auto px-4 py-6 flex flex-col md:flex-row justify-between items-center gap-6">
              <div>
                <span className="text-[10px] font-bold tracking-wider uppercase opacity-80 bg-emerald-500/30 px-2 py-1 rounded">
                  PWA Mobile Ativo 🚀
                </span>
                <h1 className="text-2xl font-black mt-1">Finanças do Red</h1>
              </div>

              <div className="flex items-center gap-3 bg-emerald-800/40 p-2 rounded-xl border border-emerald-500/20">
                <button onClick={() => navigateMonth(-1)} className="px-2 font-bold">◀</button>
                <div className="flex flex-col items-center">
                  <span className="text-xs font-bold">{formatMonthName(currentMonth)}</span>
                  <input type="month" value={currentMonth} onChange={handleMonthPickerChange} className="mt-1 text-[9px] bg-emerald-900/60 rounded px-1" />
                </div>
                <button onClick={() => navigateMonth(1)} className="px-2 font-bold">▶</button>
              </div>
            </div>

            <div className="max-w-7xl mx-auto px-4 bg-emerald-900/20">
              <div className="flex space-x-1 py-1.5">
                <button onClick={() => setActiveTab('dashboard')} className={`px-3 py-1.5 text-xs font-bold rounded-lg transition ${activeTab === 'dashboard' ? 'bg-white text-emerald-800' : 'text-emerald-100'}`}>📊 Painel</button>
                <button onClick={() => setActiveTab('spreadsheet')} className={`px-3 py-1.5 text-xs font-bold rounded-lg transition ${activeTab === 'spreadsheet' ? 'bg-white text-emerald-800' : 'text-emerald-100'}`}>🟢 Planilha</button>
                <button onClick={() => setActiveTab('backup')} className={`px-3 py-1.5 text-xs font-bold rounded-lg transition ${activeTab === 'backup' ? 'bg-white text-emerald-800' : 'text-emerald-100'}`}>💾 Segurança</button>
              </div>
            </div>
          </header>

          <main className="max-w-7xl mx-auto px-4 py-6 space-y-6">
            <section className="grid grid-cols-2 lg:grid-cols-4 gap-4">
              <div className="bg-white p-4 rounded-xl border border-slate-100 shadow-sm">
                <span className="text-[10px] uppercase font-bold text-slate-400">Renda</span>
                {isEditingIncome ? (
                  <div className="flex items-center gap-1 mt-1">
                    <input type="number" value={tempIncome} onChange={e => setTempIncome(e.target.value)} className="w-16 border rounded text-xs p-1" />
                    <button onClick={handleSaveIncome} className="text-xs bg-emerald-600 text-white px-1.5 rounded">Salvar</button>
                  </div>
                ) : (
                  <h3 className="text-base font-black flex items-center gap-2 mt-1">
                    R$ {income.toLocaleString('pt-BR')}
                    <button onClick={() => setIsEditingIncome(true)} className="text-[10px] text-emerald-600">✏️</button>
                  </h3>
                )}
              </div>

              <div className="bg-white p-4 rounded-xl border border-slate-100 shadow-sm">
                <span className="text-[10px] uppercase font-bold text-slate-400">Gastos Fixos</span>
                <h3 className="text-base font-black mt-1">R$ {totalFixed.toLocaleString('pt-BR')}</h3>
              </div>

              <div className="bg-white p-4 rounded-xl border border-slate-100 shadow-sm">
                <span className="text-[10px] uppercase font-bold text-slate-400">Gastos Pontuais</span>
                <h3 className="text-base font-black mt-1">R$ {totalOneTime.toLocaleString('pt-BR')}</h3>
              </div>

              <div className="bg-white p-4 rounded-xl border border-slate-100 shadow-sm">
                <span className="text-[10px] uppercase font-bold text-slate-400">Margem Restante</span>
                <h3 className={`text-base font-black mt-1 ${remainingBalance >= 0 ? 'text-emerald-600' : 'text-rose-600'}`}>R$ {remainingBalance.toLocaleString('pt-BR')}</h3>
              </div>
            </section>

            {activeMonthExpenses.length === 0 && (
              <div className="bg-indigo-50 border border-indigo-100 rounded-xl p-4 flex flex-col sm:flex-row justify-between items-center gap-3">
                <p className="text-xs text-indigo-900 font-semibold">Este mês está vazio! Copiar os gastos fixos recorrentes do mês anterior?</p>
                <button onClick={handleImportPreviousFixed} className="bg-indigo-600 text-white text-xs font-bold px-4 py-2 rounded-lg">🔄 Copiar Fixos</button>
              </div>
            )}

            {activeTab === 'dashboard' && (
              <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div className="bg-white p-4 rounded-xl shadow-sm border border-slate-100 lg:col-span-2 space-y-4">
                  <div className="flex justify-between items-center">
                    <h3 className="font-bold text-sm">🎯 Meta de Economia ({formatMonthName(currentMonth)})</h3>
                    {isEditingGoal ? (
                      <div className="flex gap-1">
                        <input type="number" value={tempGoal} onChange={e => setTempGoal(e.target.value)} className="w-16 text-xs border rounded p-0.5" />
                        <button onClick={handleSaveGoal} className="text-[10px] bg-emerald-600 text-white px-1.5 rounded">✔️</button>
                      </div>
                    ) : (
                      <span className="text-xs font-bold text-emerald-700 bg-emerald-50 px-2.5 py-0.5 rounded-full flex items-center gap-1">
                        R$ {savingsGoal}
                        <button onClick={() => setIsEditingGoal(true)}>✏️</button>
                      </span>
                    )}
                  </div>
                  <div className="w-full bg-slate-100 h-2 rounded-full overflow-hidden">
                    <div className="bg-emerald-500 h-full" style={{ width: `${savingsGoalPercentage}%` }}></div>
                  </div>
                  <p className="text-[10px] text-slate-500">Poupança projetada: R$ {remainingBalance.toLocaleString('pt-BR')} ({savingsGoalPercentage}% da meta alcançado).</p>
                </div>

                <div className="bg-white p-4 rounded-xl shadow-sm border border-slate-100 space-y-3">
                  <h3 className="font-bold text-sm">⚖️ Regra 50/30/20</h3>
                  <div className="space-y-2 text-[11px]">
                    <div className="flex justify-between"><span>Essenciais (Fixos): {fixedPercentageOfIncome.toFixed(0)}%</span><span className="font-semibold text-blue-600">Meta: 50%</span></div>
                    <div className="flex justify-between"><span>Estilo de Vida (Pontuais): {oneTimePercentageOfIncome.toFixed(0)}%</span><span className="font-semibold text-amber-600">Meta: 30%</span></div>
                    <div className="flex justify-between"><span>Economias (Margem): {savingsPercentageOfIncome.toFixed(0)}%</span><span className="font-semibold text-emerald-600">Meta: 20%</span></div>
                  </div>
                </div>

                <div className="bg-white p-4 rounded-xl border border-slate-100 h-fit">
                  <h3 className="font-bold text-sm mb-3">➕ Novo Lançamento</h3>
                  <form onSubmit={handleAddExpense} className="space-y-3 text-xs">
                    <input type="text" placeholder="Descrição" value={newDesc} onChange={e => setNewDesc(e.target.value)} className="w-full p-2 border rounded-lg" required />
                    <div className="grid grid-cols-2 gap-2">
                      <input type="number" step="0.01" placeholder="Valor" value={newAmount} onChange={e => setNewAmount(e.target.value)} className="p-2 border rounded-lg" required />
                      <input type="date" value={newDate} onChange={e => setNewDate(e.target.value)} min={`${currentMonth}-01`} max={`${currentMonth}-31`} className="p-2 border rounded-lg" />
                    </div>
                    <div className="grid grid-cols-2 gap-2">
                      <button type="button" onClick={() => setNewType('fixed')} className={`p-2 rounded-lg font-bold border ${newType === 'fixed' ? 'bg-blue-50 border-blue-400 text-blue-700' : 'bg-white'}`}>📌 Fixo</button>
                      <button type="button" onClick={() => setNewType('one-time')} className={`p-2 rounded-lg font-bold border ${newType === 'one-time' ? 'bg-amber-50 border-amber-400 text-amber-700' : 'bg-white'}`}>🛍️ Pontual</button>
                    </div>
                    <select value={newCategory} onChange={e => setNewCategory(e.target.value)} className="w-full p-2 border rounded-lg">
                      {DEFAULT_CATEGORIES.map(cat => <option key={cat.id} value={cat.id}>{cat.icon} {cat.name}</option>)}
                    </select>
                    <button type="submit" className="w-full py-2 bg-emerald-600 text-white font-bold rounded-lg text-xs">Registrar</button>
                  </form>
                </div>

                <div className="bg-white p-4 rounded-xl border border-slate-100 lg:col-span-2 space-y-4">
                  <div className="flex flex-col sm:flex-row justify-between gap-3">
                    <input type="text" placeholder="Pesquisar..." value={searchTerm} onChange={e => setSearchTerm(e.target.value)} className="p-2 border rounded-lg text-xs" />
                    <div className="flex gap-2">
                      <select value={filterType} onChange={e => setFilterType(e.target.value)} className="p-1 border rounded text-xs"><option value="all">Tipos</option><option value="fixed">📌 Fixo</option><option value="one-time">🛍️ Pontual</option></select>
                      <select value={filterCategory} onChange={e => setFilterCategory(e.target.value)} className="p-1 border rounded text-xs"><option value="all">Categorias</option>{DEFAULT_CATEGORIES.map(c => <option key={c.id} value={c.id}>{c.name}</option>)}</select>
                    </div>
                  </div>

                  <div className="overflow-x-auto max-h-[300px] border rounded-lg">
                    <table className="min-w-full divide-y divide-slate-100 text-xs">
                      <tbody className="bg-white divide-y divide-slate-100">
                        {filteredExpenses.map(exp => {
                          const cat = DEFAULT_CATEGORIES.find(c => c.id === exp.category) || DEFAULT_CATEGORIES[6];
                          return (
                            <tr key={exp.id} className="hover:bg-slate-50">
                              <td className="p-3">
                                <div className="font-bold">{exp.description}</div>
                                <span className="text-[9px] text-slate-400">{cat.icon} {cat.name} | {exp.date.split('-').slice(1).reverse().join('/')}</span>
                              </td>
                              <td className="p-3 text-center">{exp.type === 'fixed' ? '📌 Fixo' : '🛍️ Pontual'}</td>
                              <td className="p-3 text-right font-black">R$ {exp.amount}</td>
                              <td className="p-3 text-center"><button onClick={() => handleDeleteExpense(exp.id)} className="text-rose-400">🗑️</button></td>
                            </tr>
                          );
                        })}
                      </tbody>
                    </table>
                  </div>
                </div>
              </div>
            )}

            {activeTab === 'spreadsheet' && (
              <div className="bg-white p-4 rounded-xl shadow-sm border border-slate-100 space-y-4">
                <div className="flex flex-wrap gap-2 justify-between items-center">
                  <h3 className="font-bold text-sm">🟢 Modo Edição Direta (Planilha)</h3>
                  <div className="flex gap-2">
                    <button onClick={handleCopyToClipboardForSheets} className="bg-emerald-50 border border-emerald-200 text-emerald-700 px-3 py-1.5 rounded-lg text-xs font-bold">📋 Copiar p/ Google Sheets</button>
                    <button onClick={handleExportCSV} className="bg-slate-800 text-white px-3 py-1.5 rounded-lg text-xs font-bold">📥 Exportar CSV</button>
                  </div>
                </div>

                <div className="overflow-x-auto border rounded-xl">
                  <table className="min-w-full divide-y divide-slate-200 text-xs">
                    <thead className="bg-slate-50 font-bold">
                      <tr>
                        <th className="p-3 text-left">Descrição</th>
                        <th className="p-3 text-center">Categoria</th>
                        <th className="p-3 text-center">Tipo</th>
                        <th className="p-3 text-center">Data</th>
                        <th className="p-3 text-right">Valor</th>
                        <th className="p-3 text-center"></th>
                      </tr>
                    </thead>
                    <tbody className="bg-white divide-y">
                      {activeMonthExpenses.map(exp => (
                        <tr key={exp.id}>
                          <td className="p-2"><input type="text" value={exp.description} onChange={e => handleSpreadsheetCellChange(exp.id, 'description', e.target.value)} className="w-full bg-transparent hover:bg-slate-100 p-1 rounded" /></td>
                          <td className="p-2 text-center">
                            <select value={exp.category} onChange={e => handleSpreadsheetCellChange(exp.id, 'category', e.target.value)} className="bg-transparent hover:bg-slate-100 p-1 rounded text-slate-600">
                              {DEFAULT_CATEGORIES.map(cat => <option key={cat.id} value={cat.id}>{cat.icon} {cat.name}</option>)}
                            </select>
                          </td>
                          <td className="p-2 text-center">
                            <select value={exp.type} onChange={e => handleSpreadsheetCellChange(exp.id, 'type', e.target.value)} className="bg-transparent hover:bg-slate-100 p-1 rounded font-bold">
                              <option value="fixed">📌 Fixo</option>
                              <option value="one-time">🛍️ Pontual</option>
                            </select>
                          </td>
                          <td className="p-2 text-center"><input type="date" value={exp.date} onChange={e => handleSpreadsheetCellChange(exp.id, 'date', e.target.value)} className="bg-transparent hover:bg-slate-100 p-1 rounded" /></td>
                          <td className="p-2 text-right"><input type="number" step="0.01" value={exp.amount} onChange={e => handleSpreadsheetCellChange(exp.id, 'amount', e.target.value)} className="w-16 bg-transparent hover:bg-slate-100 p-1 rounded text-right font-bold" /></td>
                          <td className="p-2 text-center"><button onClick={() => handleDeleteExpense(exp.id)} className="text-slate-400">🗑️</button></td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              </div>
            )}

            {activeTab === 'backup' && (
              <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div className="bg-white p-4 rounded-xl border border-slate-100 space-y-3 text-xs">
                  <h3 className="font-bold text-rose-800">🛡️ Proteção de Dados</h3>
                  <p>Todos os dados ficam salvos localmente na memória interna do seu navegador do celular (`localStorage`).</p>
                  <p className="bg-amber-50 p-2 rounded text-amber-950 font-medium">⚠️ Se você apagar o cache/dados do Chrome no celular, seus dados podem sumir. Por isso, use o botão ao lado para baixar um arquivo de backup no fim de cada mês.</p>
                </div>

                <div className="bg-white p-4 rounded-xl border border-slate-100 space-y-4 text-xs">
                  <h3 className="font-bold">💾 Gestão de Cópias</h3>
                  <button onClick={handleExportBackup} className="w-full py-2 bg-emerald-600 text-white font-bold rounded-lg text-center">📥 Baixar Backup (.JSON)</button>
                  <div className="pt-2 border-t">
                    <p className="mb-2 text-slate-400">Restaurar de arquivo salvo:</p>
                    <input type="file" onChange={handleImportBackup} accept=".json" className="text-xs" />
                  </div>
                </div>
              </div>
            )}
          </main>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
