<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pro Web Calculator</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #222;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
        }

        .calculator-grid {
            display: grid;
            justify-content: center;
            align-content: center;
            /* Changed to 5 columns to fit scientific buttons */
            grid-template-columns: repeat(5, 75px);
            grid-template-rows: minmax(100px, auto) repeat(5, 75px);
            background-color: #333;
            border-radius: 15px;
            box-shadow: 0px 10px 20px rgba(0,0,0,0.5);
            overflow: hidden;
        }

        .output {
            grid-column: 1 / -1;
            background-color: #111;
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            justify-content: space-around;
            padding: 20px;
            word-wrap: break-word;
            word-break: break-all;
        }

        .output .previous-operand {
            color: rgba(255, 255, 255, 0.75);
            font-size: 1.2rem;
        }

        .output .current-operand {
            color: white;
            font-size: 2.5rem;
            font-weight: bold;
        }

        button {
            cursor: pointer;
            font-size: 1.3rem;
            border: 1px solid #444;
            outline: none;
            background-color: #444;
            color: white;
            transition: all 0.2s;
        }

        button:hover {
            background-color: #555;
        }

        .span-two {
            grid-column: span 2;
        }

        .operator {
            background-color: #fca311;
            color: black;
            font-weight: bold;
        }

        .operator:hover {
            background-color: #e5940e;
        }

        .scientific {
            background-color: #4a4a4a;
            color: #8be9fd;
            font-weight: bold;
        }
        
        .clear-btn {
            background-color: #ff5e5e;
            font-weight: bold;
            color: white;
        }
        
        .clear-btn:hover {
             background-color: #d64545;
        }
    </style>
</head>
<body>

    <div class="calculator-grid">
        <div class="output">
            <div data-previous-operand class="previous-operand"></div>
            <div data-current-operand class="current-operand"></div>
        </div>
        
        <button data-all-clear class="clear-btn span-two">AC</button>
        <button data-delete>DEL</button>
        <button data-scientific="sqrt" class="scientific">√</button>
        <button data-operation class="operator">÷</button>

        <button data-number>1</button>
        <button data-number>2</button>
        <button data-number>3</button>
        <button data-operation="^" class="scientific">^</button>
        <button data-operation class="operator">*</button>

        <button data-number>4</button>
        <button data-number>5</button>
        <button data-number>6</button>
        <button data-number>(</button> <button data-operation class="operator">+</button>

        <button data-number>7</button>
        <button data-number>8</button>
        <button data-number>9</button>
        <button data-number>)</button> <button data-operation class="operator">-</button>

        <button data-number>.</button>
        <button data-number>0</button>
        <button data-equals class="span-three operator" style="grid-column: span 3;">=</button>
    </div>

    <script>
        class Calculator {
            constructor(previousOperandTextElement, currentOperandTextElement) {
                this.previousOperandTextElement = previousOperandTextElement;
                this.currentOperandTextElement = currentOperandTextElement;
                this.clear();
            }

            clear() {
                this.currentOperand = '';
                this.previousOperand = '';
                this.operation = undefined;
            }

            delete() {
                this.currentOperand = this.currentOperand.toString().slice(0, -1);
            }

            appendNumber(number) {
                if (number === '.' && this.currentOperand.includes('.')) return;
                this.currentOperand = this.currentOperand.toString() + number.toString();
            }

            chooseOperation(operation) {
                if (this.currentOperand === '') return;
                if (this.previousOperand !== '') {
                    this.compute();
                }
                this.operation = operation;
                this.previousOperand = this.currentOperand;
                this.currentOperand = '';
            }

            // New method for immediate scientific operations (like Square Root)
            computeScientific(type) {
                let current = parseFloat(this.currentOperand);
                if (isNaN(current)) return;
                
                let result;
                switch (type) {
                    case 'sqrt':
                        if (current < 0) return; // Basic error prevention
                        result = Math.sqrt(current);
                        break;
                    default:
                        return;
                }
                this.currentOperand = result;
            }

            compute() {
                let computation;
                const prev = parseFloat(this.previousOperand);
                const current = parseFloat(this.currentOperand);
                if (isNaN(prev) || isNaN(current)) return;
                
                switch (this.operation) {
                    case '+':
                        computation = prev + current;
                        break;
                    case '-':
                        computation = prev - current;
                        break;
                    case '*':
                        computation = prev * current;
                        break;
                    case '÷':
                        computation = prev / current;
                        break;
                    case '^': // Power Logic
                        computation = Math.pow(prev, current);
                        break;
                    default:
                        return;
                }
                this.currentOperand = computation;
                this.operation = undefined;
                this.previousOperand = '';
            }

            getDisplayNumber(number) {
                const stringNumber = number.toString();
                const integerDigits = parseFloat(stringNumber.split('.')[0]);
                const decimalDigits = stringNumber.split('.')[1];
                let integerDisplay;
                if (isNaN(integerDigits)) {
                    integerDisplay = '';
                } else {
                    integerDisplay = integerDigits.toLocaleString('en', { maximumFractionDigits: 0 });
                }
                if (decimalDigits != null) {
                    return `${integerDisplay}.${decimalDigits}`;
                } else {
                    return integerDisplay;
                }
            }

            updateDisplay() {
                this.currentOperandTextElement.innerText = 
                    this.getDisplayNumber(this.currentOperand);
                if (this.operation != null) {
                    this.previousOperandTextElement.innerText = 
                        `${this.getDisplayNumber(this.previousOperand)} ${this.operation}`;
                } else {
                    this.previousOperandTextElement.innerText = '';
                }
            }
        }

        const numberButtons = document.querySelectorAll('[data-number]');
        const operationButtons = document.querySelectorAll('[data-operation]');
        const scientificButtons = document.querySelectorAll('[data-scientific]'); // New selector
        const equalsButton = document.querySelector('[data-equals]');
        const deleteButton = document.querySelector('[data-delete]');
        const allClearButton = document.querySelector('[data-all-clear]');
        const previousOperandTextElement = document.querySelector('[data-previous-operand]');
        const currentOperandTextElement = document.querySelector('[data-current-operand]');

        const calculator = new Calculator(previousOperandTextElement, currentOperandTextElement);

        // Click Listeners
        numberButtons.forEach(button => {
            button.addEventListener('click', () => {
                calculator.appendNumber(button.innerText);
                calculator.updateDisplay();
            });
        });

        operationButtons.forEach(button => {
            button.addEventListener('click', () => {
                calculator.chooseOperation(button.innerText);
                calculator.updateDisplay();
            });
        });

        scientificButtons.forEach(button => {
            button.addEventListener('click', () => {
                // If it's a binary op like ^, treat as operation, else immediate
                if (button.innerText === '^') {
                    calculator.chooseOperation('^');
                } else {
                    calculator.computeScientific(button.dataset.scientific);
                }
                calculator.updateDisplay();
            });
        });

        equalsButton.addEventListener('click', button => {
            calculator.compute();
            calculator.updateDisplay();
        });

        allClearButton.addEventListener('click', button => {
            calculator.clear();
            calculator.updateDisplay();
        });

        deleteButton.addEventListener('click', button => {
            calculator.delete();
            calculator.updateDisplay();
        });

        // --- NEW: Keyboard Support Logic ---
        document.addEventListener('keydown', function(event) {
            let key = event.key;
            
            // Number mapping
            if ((key >= 0 && key <= 9) || key === '.') {
                calculator.appendNumber(key);
                calculator.updateDisplay();
            }
            // Operator mapping
            if (key === '+' || key === '-' || key === '*' || key === '/') {
                calculator.chooseOperation(key);
                calculator.updateDisplay();
            }
            // Power mapping (Shift+6 usually)
            if (key === '^') {
                calculator.chooseOperation('^');
                calculator.updateDisplay();
            }
            // Enter or = for equals
            if (key === 'Enter' || key === '=') {
                event.preventDefault(); // prevents Enter from "clicking" the last focused button
                calculator.compute();
                calculator.updateDisplay();
            }
            // Backspace for Delete
            if (key === 'Backspace') {
                calculator.delete();
                calculator.updateDisplay();
            }
            // Escape for All Clear
            if (key === 'Escape') {
                calculator.clear();
                calculator.updateDisplay();
            }
        });
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Calculator</title>
    <style>
        /* CSS VARIABLES: This handles the color flipping */
        :root {
            --bg-body: #222;
            --bg-calc: #333;
            --bg-screen: #111;
            --bg-btn: #444;
            --bg-btn-hover: #555;
            --text-main: #fff;
            --text-secondary: rgba(255, 255, 255, 0.75);
            --shadow: 0px 10px 20px rgba(0,0,0,0.5);
            --accent-op: #fca311;
            --accent-op-hover: #e5940e;
            --accent-sci: #4a4a4a;
            --text-sci: #8be9fd;
            --accent-clear: #ff5e5e;
            --accent-clear-hover: #d64545;
        }

        /* Light Theme Overrides */
        [data-theme="light"] {
            --bg-body: #e0e5ec;
            --bg-calc: #f0f0f3;
            --bg-screen: #e0e5ec;
            --bg-btn: #ffffff;
            --bg-btn-hover: #f9f9f9;
            --text-main: #333;
            --text-secondary: rgba(0, 0, 0, 0.55);
            --shadow: 9px 9px 16px rgb(163,177,198,0.6), -9px -9px 16px rgba(255,255,255, 0.5);
            --accent-op: #ffc45e;
            --accent-op-hover: #ffcf7b;
            --accent-sci: #d1d9e6;
            --text-sci: #007bff;
            --accent-clear: #ff8c8c;
            --accent-clear-hover: #ff9e9e;
        }

        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: var(--bg-body);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            transition: background-color 0.3s;
        }

        /* Toggle Switch Styling */
        .theme-switch-wrapper {
            display: flex;
            align-items: center;
            position: absolute;
            top: 20px;
            right: 20px;
            color: var(--text-main);
        }

        .theme-switch {
            display: inline-block;
            height: 34px;
            position: relative;
            width: 60px;
            margin-left: 10px;
        }

        .theme-switch input {
            display: none;
        }

        .slider {
            background-color: #ccc;
            bottom: 0;
            cursor: pointer;
            left: 0;
            position: absolute;
            right: 0;
            top: 0;
            transition: .4s;
            border-radius: 34px;
        }

        .slider:before {
            background-color: #fff;
            bottom: 4px;
            content: "";
            height: 26px;
            left: 4px;
            position: absolute;
            transition: .4s;
            width: 26px;
            border-radius: 50%;
        }

        input:checked + .slider {
            background-color: var(--accent-op);
        }

        input:checked + .slider:before {
            transform: translateX(26px);
        }

        /* Calculator Grid */
        .calculator-grid {
            display: grid;
            justify-content: center;
            align-content: center;
            grid-template-columns: repeat(5, 75px);
            grid-template-rows: minmax(100px, auto) repeat(5, 75px);
            background-color: var(--bg-calc);
            border-radius: 15px;
            box-shadow: var(--shadow);
            overflow: hidden;
            transition: background-color 0.3s, box-shadow 0.3s;
        }

        .output {
            grid-column: 1 / -1;
            background-color: var(--bg-screen);
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            justify-content: space-around;
            padding: 20px;
            word-wrap: break-word;
            word-break: break-all;
            transition: background-color 0.3s;
        }

        .output .previous-operand {
            color: var(--text-secondary);
            font-size: 1.2rem;
        }

        .output .current-operand {
            color: var(--text-main);
            font-size: 2.5rem;
            font-weight: bold;
        }

        button {
            cursor: pointer;
            font-size: 1.3rem;
            border: 1px solid rgba(0,0,0,0.05);
            outline: none;
            background-color: var(--bg-btn);
            color: var(--text-main);
            transition: all 0.2s;
        }

        button:hover {
            background-color: var(--bg-btn-hover);
        }

        .span-two { grid-column: span 2; }
        .span-three { grid-column: span 3; }

        .operator {
            background-color: var(--accent-op);
            color: black;
            font-weight: bold;
        }
        .operator:hover { background-color: var(--accent-op-hover); }

        .scientific {
            background-color: var(--accent-sci);
            color: var(--text-sci);
            font-weight: bold;
        }
        
        .clear-btn {
            background-color: var(--accent-clear);
            font-weight: bold;
            color: white;
        }
        .clear-btn:hover { background-color: var(--accent-clear-hover); }

    </style>
</head>
<body>

    <div class="theme-switch-wrapper">
        <span id="theme-label">Dark Mode</span>
        <label class="theme-switch" for="checkbox">
            <input type="checkbox" id="checkbox" />
            <div class="slider round"></div>
        </label>
    </div>

    <div class="calculator-grid">
        <div class="output">
            <div data-previous-operand class="previous-operand"></div>
            <div data-current-operand class="current-operand"></div>
        </div>
        
        <button data-all-clear class="clear-btn span-two">AC</button>
        <button data-delete>DEL</button>
        <button data-scientific="sqrt" class="scientific">√</button>
        <button data-operation class="operator">÷</button>

        <button data-number>1</button>
        <button data-number>2</button>
        <button data-number>3</button>
        <button data-operation="^" class="scientific">^</button>
        <button data-operation class="operator">*</button>

        <button data-number>4</button>
        <button data-number>5</button>
        <button data-number>6</button>
        <button data-number>(</button>
        <button data-operation class="operator">+</button>

        <button data-number>7</button>
        <button data-number>8</button>
        <button data-number>9</button>
        <button data-number>)</button>
        <button data-operation class="operator">-</button>

        <button data-number>.</button>
        <button data-number>0</button>
        <button data-equals class="span-three operator">=</button>
    </div>

    <script>
        // --- 1. Theme Logic ---
        const toggleSwitch = document.querySelector('.theme-switch input[type="checkbox"]');
        const currentTheme = localStorage.getItem('theme');
        const themeLabel = document.getElementById('theme-label');

        if (currentTheme) {
            document.documentElement.setAttribute('data-theme', currentTheme);
            if (currentTheme === 'light') {
                toggleSwitch.checked = true;
                themeLabel.innerText = "Light Mode";
            }
        }

        function switchTheme(e) {
            if (e.target.checked) {
                document.documentElement.setAttribute('data-theme', 'light');
                localStorage.setItem('theme', 'light');
                themeLabel.innerText = "Light Mode";
            } else {
                document.documentElement.setAttribute('data-theme', 'dark');
                localStorage.setItem('theme', 'dark');
                themeLabel.innerText = "Dark Mode";
            }
        }

        toggleSwitch.addEventListener('change', switchTheme, false);

        // --- 2. Calculator Logic (Same as Pro version) ---
        class Calculator {
            constructor(previousOperandTextElement, currentOperandTextElement) {
                this.previousOperandTextElement = previousOperandTextElement;
                this.currentOperandTextElement = currentOperandTextElement;
                this.clear();
            }

            clear() {
                this.currentOperand = '';
                this.previousOperand = '';
                this.operation = undefined;
            }

            delete() {
                this.currentOperand = this.currentOperand.toString().slice(0, -1);
            }

            appendNumber(number) {
                if (number === '.' && this.currentOperand.includes('.')) return;
                this.currentOperand = this.currentOperand.toString() + number.toString();
            }

            chooseOperation(operation) {
                if (this.currentOperand === '') return;
                if (this.previousOperand !== '') {
                    this.compute();
                }
                this.operation = operation;
                this.previousOperand = this.currentOperand;
                this.currentOperand = '';
            }

            computeScientific(type) {
                let current = parseFloat(this.currentOperand);
                if (isNaN(current)) return;
                let result;
                switch (type) {
                    case 'sqrt':
                        if (current < 0) return;
                        result = Math.sqrt(current);
                        break;
                    default: return;
                }
                this.currentOperand = result;
            }

            compute() {
                let computation;
                const prev = parseFloat(this.previousOperand);
                const current = parseFloat(this.currentOperand);
                if (isNaN(prev) || isNaN(current)) return;
                switch (this.operation) {
                    case '+': computation = prev + current; break;
                    case '-': computation = prev - current; break;
                    case '*': computation = prev * current; break;
                    case '÷': computation = prev / current; break;
                    case '^': computation = Math.pow(prev, current); break;
                    default: return;
                }
                this.currentOperand = computation;
                this.operation = undefined;
                this.previousOperand = '';
            }

            getDisplayNumber(number) {
                const stringNumber = number.toString();
                const integerDigits = parseFloat(stringNumber.split('.')[0]);
                const decimalDigits = stringNumber.split('.')[1];
                let integerDisplay;
                if (isNaN(integerDigits)) {
                    integerDisplay = '';
                } else {
                    integerDisplay = integerDigits.toLocaleString('en', { maximumFractionDigits: 0 });
                }
                if (decimalDigits != null) {
                    return `${integerDisplay}.${decimalDigits}`;
                } else {
                    return integerDisplay;
                }
            }

            updateDisplay() {
                this.currentOperandTextElement.innerText = this.getDisplayNumber(this.currentOperand);
                if (this.operation != null) {
                    this.previousOperandTextElement.innerText = 
                        `${this.getDisplayNumber(this.previousOperand)} ${this.operation}`;
                } else {
                    this.previousOperandTextElement.innerText = '';
                }
            }
        }

        const numberButtons = document.querySelectorAll('[data-number]');
        const operationButtons = document.querySelectorAll('[data-operation]');
        const scientificButtons = document.querySelectorAll('[data-scientific]');
        const equalsButton = document.querySelector('[data-equals]');
        const deleteButton = document.querySelector('[data-delete]');
        const allClearButton = document.querySelector('[data-all-clear]');
        const previousOperandTextElement = document.querySelector('[data-previous-operand]');
        const currentOperandTextElement = document.querySelector('[data-current-operand]');

        const calculator = new Calculator(previousOperandTextElement, currentOperandTextElement);

        numberButtons.forEach(button => {
            button.addEventListener('click', () => {
                calculator.appendNumber(button.innerText);
                calculator.updateDisplay();
            });
        });

        operationButtons.forEach(button => {
            button.addEventListener('click', () => {
                calculator.chooseOperation(button.innerText);
                calculator.updateDisplay();
            });
        });

        scientificButtons.forEach(button => {
            button.addEventListener('click', () => {
                if (button.innerText === '^') {
                    calculator.chooseOperation('^');
                } else {
                    calculator.computeScientific(button.dataset.scientific);
                }
                calculator.updateDisplay();
            });
        });

        equalsButton.addEventListener('click', button => {
            calculator.compute();
            calculator.updateDisplay();
        });

        allClearButton.addEventListener('click', button => {
            calculator.clear();
            calculator.updateDisplay();
        });

        deleteButton.addEventListener('click', button => {
            calculator.delete();
            calculator.updateDisplay();
        });

        document.addEventListener('keydown', function(event) {
            let key = event.key;
            if ((key >= 0 && key <= 9) || key === '.') { calculator.appendNumber(key); calculator.updateDisplay(); }
            if (key === '+' || key === '-' || key === '*' || key === '/') { calculator.chooseOperation(key); calculator.updateDisplay(); }
            if (key === '^') { calculator.chooseOperation('^'); calculator.updateDisplay(); }
            if (key === 'Enter' || key === '=') { event.preventDefault(); calculator.compute(); calculator.updateDisplay(); }
            if (key === 'Backspace') { calculator.delete(); calculator.updateDisplay(); }
            if (key === 'Escape') { calculator.clear(); calculator.updateDisplay(); }
        });
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Calculator with History</title>
    <style>
        :root {
            --bg-body: #1a1a1a;
            --bg-calc: #333;
            --bg-screen: #111;
            --bg-btn: #444;
            --bg-btn-hover: #555;
            --text-main: #fff;
            --text-secondary: rgba(255, 255, 255, 0.75);
            --shadow: 0px 10px 30px rgba(0,0,0,0.5);
            --accent-op: #fca311;
            --accent-op-hover: #e5940e;
            --accent-sci: #4a4a4a;
            --text-sci: #8be9fd;
            --accent-clear: #ff5e5e;
            --bg-history: #252525;
        }

        [data-theme="light"] {
            --bg-body: #d1d9e6;
            --bg-calc: #f0f0f3;
            --bg-screen: #e0e5ec;
            --bg-btn: #ffffff;
            --bg-btn-hover: #f9f9f9;
            --text-main: #333;
            --text-secondary: rgba(0, 0, 0, 0.55);
            --shadow: 9px 9px 16px rgb(163,177,198,0.6);
            --accent-op: #ffc45e;
            --accent-op-hover: #ffcf7b;
            --accent-sci: #d1d9e6;
            --text-sci: #007bff;
            --accent-clear: #ff8c8c;
            --bg-history: #f0f0f3;
        }

        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: var(--bg-body);
            font-family: 'Segoe UI', sans-serif;
            margin: 0;
            transition: 0.3s;
        }

        /* Container for Calc + History */
        .main-container {
            display: flex;
            gap: 20px;
            padding: 20px;
            max-width: 900px;
            width: 95%;
            flex-wrap: wrap;
            justify-content: center;
        }

        /* Theme Toggle */
        .theme-switch-wrapper {
            position: absolute;
            top: 20px;
            right: 20px;
            color: var(--text-main);
            display: flex;
            align-items: center;
        }

        /* Calculator Styles */
        .calculator-grid {
            display: grid;
            grid-template-columns: repeat(5, 70px);
            grid-template-rows: minmax(120px, auto) repeat(5, 70px);
            background-color: var(--bg-calc);
            border-radius: 20px;
            box-shadow: var(--shadow);
            overflow: hidden;
        }

        .output {
            grid-column: 1 / -1;
            background-color: var(--bg-screen);
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            justify-content: space-around;
            padding: 15px;
            word-wrap: break-word;
        }

        .output .previous-operand { color: var(--text-secondary); font-size: 1.1rem; }
        .output .current-operand { color: var(--text-main); font-size: 2.2rem; font-weight: bold; }

        button {
            cursor: pointer;
            font-size: 1.2rem;
            border: 0.5px solid rgba(0,0,0,0.1);
            background-color: var(--bg-btn);
            color: var(--text-main);
            transition: 0.2s;
        }
        button:hover { background-color: var(--bg-btn-hover); }
        .operator { background-color: var(--accent-op); color: #000; }
        .scientific { background-color: var(--accent-sci); color: var(--text-sci); }
        .clear-btn { background-color: var(--accent-clear); color: #fff; }

        /* History Sidebar */
        .history-panel {
            width: 250px;
            background-color: var(--bg-history);
            border-radius: 20px;
            box-shadow: var(--shadow);
            display: flex;
            flex-direction: column;
            padding: 15px;
            max-height: 470px;
        }

        .history-panel h3 {
            margin: 0 0 10px 0;
            color: var(--text-main);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .history-list {
            flex-grow: 1;
            overflow-y: auto;
            margin-bottom: 10px;
        }

        .history-item {
            padding: 10px;
            border-bottom: 1px solid rgba(127,127,127,0.2);
            cursor: pointer;
            transition: 0.2s;
            color: var(--text-secondary);
            font-size: 0.9rem;
            text-align: right;
        }

        .history-item:hover { background-color: rgba(255,255,255,0.05); }

        .history-item span {
            display: block;
            color: var(--text-main);
            font-size: 1.1rem;
            font-weight: bold;
        }

        .clear-history {
            background: none;
            border: 1px solid var(--accent-clear);
            color: var(--accent-clear);
            padding: 5px 10px;
            border-radius: 5px;
            font-size: 0.8rem;
        }
        
        .clear-history:hover { background: var(--accent-clear); color: white; }

        /* Responsive Mobile */
        @media (max-width: 600px) {
            .calculator-grid { grid-template-columns: repeat(5, 60px); grid-template-rows: minmax(100px, auto) repeat(5, 60px); }
            .history-panel { width: 100%; }
        }
    </style>
</head>
<body>

    <div class="theme-switch-wrapper">
        <input type="checkbox" id="checkbox" style="margin-right: 10px;">
        <label for="checkbox">Light Mode</label>
    </div>

    <div class="main-container">
        <div class="calculator-grid">
            <div class="output">
                <div data-previous-operand class="previous-operand"></div>
                <div data-current-operand class="current-operand"></div>
            </div>
            <button data-all-clear class="clear-btn style="grid-column: span 2;">AC</button>
            <button data-delete>DEL</button>
            <button data-scientific="sqrt" class="scientific">√</button>
            <button data-operation class="operator">÷</button>

            <button data-number>7</button><button data-number>8</button><button data-number>9</button>
            <button data-operation="^" class="scientific">^</button>
            <button data-operation class="operator">*</button>

            <button data-number>4</button><button data-number>5</button><button data-number>6</button>
            <button class="scientific">(</button>
            <button data-operation class="operator">+</button>

            <button data-number>1</button><button data-number>2</button><button data-number>3</button>
            <button class="scientific">)</button>
            <button data-operation class="operator">-</button>

            <button data-number>.</button><button data-number>0</button>
            <button data-equals style="grid-column: span 3;" class="operator">=</button>
        </div>

        <div class="history-panel">
            <h3>History <button class="clear-history" id="clear-hist">Clear</button></h3>
            <div class="history-list" id="hist-list">
                </div>
        </div>
    </div>

    <script>
        class Calculator {
            constructor(prevEl, currEl, histEl) {
                this.prevEl = prevEl;
                this.currEl = currEl;
                this.histEl = histEl;
                this.clear();
            }

            clear() { this.current = ''; this.previous = ''; this.operation = undefined; }
            delete() { this.current = this.current.toString().slice(0, -1); }
            append(num) { 
                if (num === '.' && this.current.includes('.')) return;
                this.current = this.current.toString() + num.toString(); 
            }
            chooseOp(op) {
                if (this.current === '') return;
                if (this.previous !== '') this.compute();
                this.operation = op;
                this.previous = this.current;
                this.current = '';
            }

            compute() {
                let res;
                const prev = parseFloat(this.previous);
                const curr = parseFloat(this.current);
                if (isNaN(prev) || isNaN(curr)) return;
                switch (this.operation) {
                    case '+': res = prev + curr; break;
                    case '-': res = prev - curr; break;
                    case '*': res = prev * curr; break;
                    case '÷': res = prev / curr; break;
                    case '^': res = Math.pow(prev, curr); break;
                    default: return;
                }
                
                this.addToHistory(`${this.previous} ${this.operation} ${this.current}`, res);
                this.current = res;
                this.operation = undefined;
                this.previous = '';
            }

            computeSci(type) {
                const curr = parseFloat(this.current);
                if (isNaN(curr)) return;
                let res = type === 'sqrt' ? Math.sqrt(curr) : curr;
                this.addToHistory(`√(${this.current})`, res);
                this.current = res;
            }

            addToHistory(equation, result) {
                const div = document.createElement('div');
                div.classList.add('history-item');
                div.innerHTML = `${equation} = <span>${result}</span>`;
                div.onclick = () => { this.current = result.toString(); this.updateDisplay(); };
                this.histEl.prepend(div);
            }

            updateDisplay() {
                this.currEl.innerText = this.current;
                this.prevEl.innerText = this.operation ? `${this.previous} ${this.operation}` : '';
            }
        }

        // Init
        const calc = new Calculator(
            document.querySelector('[data-previous-operand]'),
            document.querySelector('[data-current-operand]'),
            document.getElementById('hist-list')
        );

        // Buttons
        document.querySelectorAll('[data-number]').forEach(b => b.onclick = () => { calc.append(b.innerText); calc.updateDisplay(); });
        document.querySelectorAll('[data-operation]').forEach(b => b.onclick = () => { calc.chooseOp(b.innerText); calc.updateDisplay(); });
        document.querySelector('[data-equals]').onclick = () => { calc.compute(); calc.updateDisplay(); };
        document.querySelector('[data-all-clear]').onclick = () => { calc.clear(); calc.updateDisplay(); };
        document.querySelector('[data-delete]').onclick = () => { calc.delete(); calc.updateDisplay(); };
        document.querySelector('[data-scientific="sqrt"]').onclick = () => { calc.computeSci('sqrt'); calc.updateDisplay(); };
        document.getElementById('clear-hist').onclick = () => document.getElementById('hist-list').innerHTML = '';

        // Theme Toggle
        const toggle = document.getElementById('checkbox');
        toggle.onchange = (e) => {
            document.documentElement.setAttribute('data-theme', e.target.checked ? 'light' : 'dark');
        };
    </script>
</body>
</html>
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Master Utility: Calc & Convert</title>
    <style>
        :root {
            --bg-body: #121212;
            --bg-card: #1e1e1e;
            --bg-input: #2d2d2d;
            --text-main: #ffffff;
            --text-dim: #b0b0b0;
            --accent: #bb86fc;
            --accent-op: #03dac6;
            --shadow: 0 8px 32px rgba(0,0,0,0.3);
        }

        [data-theme="light"] {
            --bg-body: #f5f7fa;
            --bg-card: #ffffff;
            --bg-input: #f0f0f0;
            --text-main: #2c3e50;
            --text-dim: #7f8c8d;
            --accent: #3498db;
            --accent-op: #e67e22;
            --shadow: 0 8px 32px rgba(0,0,0,0.1);
        }

        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--bg-body);
            color: var(--text-main);
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            transition: 0.3s;
        }

        .header-tools {
            display: flex;
            gap: 15px;
            margin: 20px;
        }

        .btn-tool {
            padding: 10px 20px;
            border-radius: 30px;
            border: none;
            cursor: pointer;
            background: var(--bg-card);
            color: var(--text-main);
            box-shadow: var(--shadow);
            font-weight: bold;
        }

        .btn-tool.active {
            background: var(--accent);
            color: #000;
        }

        .main-app {
            display: flex;
            gap: 20px;
            padding: 20px;
            flex-wrap: wrap;
            justify-content: center;
        }

        /* --- Calculator --- */
        .calculator-grid {
            display: grid;
            grid-template-columns: repeat(5, 70px);
            grid-template-rows: minmax(120px, auto) repeat(5, 70px);
            background-color: var(--bg-card);
            border-radius: 24px;
            box-shadow: var(--shadow);
            overflow: hidden;
        }

        #calculator-view.hidden { display: none; }

        .output {
            grid-column: 1 / -1;
            background: rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            justify-content: center;
            padding: 20px;
        }

        .output .prev { color: var(--text-dim); font-size: 1rem; }
        .output .curr { color: var(--text-main); font-size: 2.2rem; font-weight: bold; }

        .calculator-grid button {
            border: 0.5px solid rgba(127,127,127,0.1);
            background: var(--bg-card);
            color: var(--text-main);
            font-size: 1.2rem;
            cursor: pointer;
        }

        .calculator-grid button:hover { background: var(--bg-input); }
        .op { color: var(--accent-op); font-weight: bold; }
        .equals { background: var(--accent) !important; color: #000 !important; }

        /* --- Converter --- */
        #converter-view {
            background: var(--bg-card);
            padding: 30px;
            border-radius: 24px;
            box-shadow: var(--shadow);
            width: 350px;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        #converter-view.hidden { display: none; }

        .conv-group { display: flex; flex-direction: column; gap: 8px; }

        select, input.conv-input {
            padding: 12px;
            border-radius: 12px;
            border: 1px solid rgba(127,127,127,0.3);
            background: var(--bg-input);
            color: var(--text-main);
            font-size: 1rem;
        }

        /* --- History --- */
        .history-pane {
            width: 250px;
            background: var(--bg-card);
            border-radius: 24px;
            padding: 20px;
            box-shadow: var(--shadow);
            max-height: 540px;
            overflow-y: auto;
        }

        .hist-item {
            padding: 10px;
            border-bottom: 1px solid rgba(127,127,127,0.1);
            font-size: 0.9rem;
            text-align: right;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <div class="header-tools">
        <button class="btn-tool active" onclick="showView('calculator')">Calculator</button>
        <button class="btn-tool" onclick="showView('converter')">Converter</button>
        <button class="btn-tool" onclick="toggleTheme()">🌓 Theme</button>
    </div>

    <div class="main-app">
        <div id="calculator-view" class="calculator-grid">
            <div class="output">
                <div class="prev" id="prev-op"></div>
                <div class="curr" id="curr-op">0</div>
            </div>
            <button onclick="clearAll()" style="grid-column: span 2;" class="op">AC</button>
            <button onclick="del()">DEL</button>
            <button onclick="setSci('sqrt')" class="op">√</button>
            <button onclick="setOp('/')" class="op">÷</button>
            <button onclick="addNum('7')">7</button><button onclick="addNum('8')">8</button><button onclick="addNum('9')">9</button>
            <button onclick="setOp('^')" class="op">^</button>
            <button onclick="setOp('*')" class="op">×</button>
            <button onclick="addNum('4')">4</button><button onclick="addNum('5')">5</button><button onclick="addNum('6')">6</button>
            <button onclick="addNum('(')">(</button><button onclick="setOp('+')" class="op">+</button>
            <button onclick="addNum('1')">1</button><button onclick="addNum('2')">2</button><button onclick="addNum('3')">3</button>
            <button onclick="addNum(')')">)</button><button onclick="setOp('-')" class="op">-</button>
            <button onclick="addNum('.')">.</button><button onclick="addNum('0')">0</button>
            <button onclick="solve()" style="grid-column: span 3;" class="equals">=</button>
        </div>

        <div id="converter-view" class="hidden">
            <h2>Unit Converter</h2>
            <div class="conv-group">
                <label>Category</label>
                <select id="conv-cat" onchange="updateUnits()">
                    <option value="length">Length (m, ft, in)</option>
                    <option value="weight">Weight (kg, lb)</option>
                    <option value="temp">Temperature (C, F)</option>
                </select>
            </div>
            <div class="conv-group">
                <input type="number" id="unit-from-val" class="conv-input" value="1" oninput="convert()">
                <select id="unit-from-type" onchange="convert()"></select>
            </div>
            <div style="text-align: center; font-weight: bold;">=</div>
            <div class="conv-group">
                <input type="number" id="unit-to-val" class="conv-input" readonly>
                <select id="unit-to-type" onchange="convert()"></select>
            </div>
        </div>

        <div class="history-pane">
            <h4 style="margin-top: 0;">History</h4>
            <div id="hist-list"></div>
        </div>
    </div>

    <script>
        // --- NAVIGATION & THEME ---
        function showView(view) {
            document.getElementById('calculator-view').classList.toggle('hidden', view !== 'calculator');
            document.getElementById('converter-view').classList.toggle('hidden', view !== 'converter');
            document.querySelectorAll('.btn-tool').forEach(b => b.classList.remove('active'));
            event.target.classList.add('active');
        }

        function toggleTheme() {
            const theme = document.documentElement.getAttribute('data-theme') === 'dark' ? 'light' : 'dark';
            document.documentElement.setAttribute('data-theme', theme);
        }

        // --- CALCULATOR LOGIC ---
        let currentInput = '';
        let previousInput = '';
        let activeOp = undefined;

        const currDisplay = document.getElementById('curr-op');
        const prevDisplay = document.getElementById('prev-op');
        const histList = document.getElementById('hist-list');

        function addNum(n) { currentInput += n; updateUI(); }
        function clearAll() { currentInput = ''; previousInput = ''; activeOp = undefined; updateUI(); }
        function del() { currentInput = currentInput.slice(0, -1); updateUI(); }
        
        function setOp(op) {
            if (currentInput === '') return;
            if (previousInput !== '') solve();
            activeOp = op;
            previousInput = currentInput;
            currentInput = '';
            updateUI();
        }

        function solve() {
            let result;
            const p = parseFloat(previousInput);
            const c = parseFloat(currentInput);
            if (isNaN(p) || isNaN(c)) return;
            switch(activeOp) {
                case '+': result = p + c; break;
                case '-': result = p - c; break;
                case '*': result = p * c; break;
                case '/': result = p / c; break;
                case '^': result = Math.pow(p, c); break;
            }
            addHist(`${previousInput} ${activeOp} ${currentInput}`, result);
            currentInput = result.toString();
            activeOp = undefined;
            previousInput = '';
            updateUI();
        }

        function setSci(type) {
            if (currentInput === '') return;
            let result = Math.sqrt(parseFloat(currentInput));
            addHist(`√(${currentInput})`, result);
            currentInput = result.toString();
            updateUI();
        }

        function updateUI() {
            currDisplay.innerText = currentInput || '0';
            prevDisplay.innerText = activeOp ? `${previousInput} ${activeOp}` : '';
        }

        function addHist(eq, res) {
            const div = document.createElement('div');
            div.className = 'hist-item';
            div.innerHTML = `${eq} = <br><strong>${res}</strong>`;
            div.onclick = () => { currentInput = res.toString(); updateUI(); };
            histList.prepend(div);
        }

        // --- CONVERTER LOGIC ---
        const units = {
            length: { m: 1, ft: 3.28084, in: 39.3701 },
            weight: { kg: 1, lb: 2.20462 },
            temp: { c: 'c', f: 'f' }
        };

        function updateUnits() {
            const cat = document.getElementById('conv-cat').value;
            const from = document.getElementById('unit-from-type');
            const to = document.getElementById('unit-to-type');
            from.innerHTML = to.innerHTML = '';
            Object.keys(units[cat]).forEach(u => {
                from.add(new Option(u.toUpperCase(), u));
                to.add(new Option(u.toUpperCase(), u));
            });
            convert();
        }

        function convert() {
            const cat = document.getElementById('conv-cat').value;
            const val = parseFloat(document.getElementById('unit-from-val').value);
            const from = document.getElementById('unit-from-type').value;
            const to = document.getElementById('unit-to-type').value;
            let result;

            if (cat === 'temp') {
                if (from === to) result = val;
                else if (from === 'c') result = (val * 9/5) + 32;
                else result = (val - 32) * 5/9;
            } else {
                result = val * (units[cat][to] / units[cat][from]);
            }
            document.getElementById('unit-to-val').value = result.toFixed(4);
        }

        updateUnits(); // Init converter
    </script>
</body>
</html>

