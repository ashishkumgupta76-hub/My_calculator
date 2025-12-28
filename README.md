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
