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
