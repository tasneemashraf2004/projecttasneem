# projectCalculator
from graphics import Canvas
import time

# Constants
CANVAS_WIDTH = 400
CANVAS_HEIGHT = 600
BUTTON_SIZE = 80
BUTTON_MARGIN = 10
DISPLAY_HEIGHT = 120

class GraphicalCalculator:
    def __init__(self):
        self.canvas = Canvas(CANVAS_WIDTH, CANVAS_HEIGHT)
        
        
        # Calculator state
        self.current_input = "0"
        self.stored_value = None
        self.current_operation = None
        self.reset_input_next = False
        self.last_click_time = 0
        
        # Colors
        self.display_bg = "#acfffc"
        self.button_bg = "pink"
        self.operation_bg = "#fdcb6e"
        self.equals_bg = "#00b894"
        self.clear_bg = "#ff6163"
        self.text_color = "#363737"
        
        # Store button positions
        self.button_areas = []
        
        self.create_display()
        self.create_buttons()
    
    def create_display(self):
        """Create the calculator display area"""
        self.display = self.canvas.create_rectangle(
            0, 0, CANVAS_WIDTH, DISPLAY_HEIGHT, self.display_bg
        )
        self.display_text = self.canvas.create_text(
            CANVAS_WIDTH - 20, DISPLAY_HEIGHT - 40, 
            text=self.current_input, 
            anchor="e",
            font="Arial 60 bold",  # LARGER DISPLAY TEXT
            color=self.text_color
        )
    
    def create_buttons(self):
        """Create all calculator buttons"""
        button_labels = [
            '7', '8', '9', '/',
            '4', '5', '6', '*',
            '1', '2', '3', '-',
            'C', '0', '=', '+'
        ]
        
        for i, label in enumerate(button_labels):
            row = i // 4
            col = i % 4
            
            x1 = col * (BUTTON_SIZE + BUTTON_MARGIN) + BUTTON_MARGIN
            y1 = row * (BUTTON_SIZE + BUTTON_MARGIN) + DISPLAY_HEIGHT + BUTTON_MARGIN
            x2 = x1 + BUTTON_SIZE
            y2 = y1 + BUTTON_SIZE
            
            # Store button area and label
            self.button_areas.append(((x1, y1, x2, y2), label))
            
            # Determine button color
            if label in ['+', '-', '*', '/']:
                color = self.operation_bg
            elif label == '=':
                color = self.equals_bg
            elif label == 'C':
                color = self.clear_bg
            else:
                color = self.button_bg
            
            # Create button with LARGER TEXT
            self.canvas.create_rectangle(x1, y1, x2, y2, color)
            self.canvas.create_text(
                (x1 + x2)//2, (y1 + y2)//2,
                text=label,
                font="Arial 36 bold",  # LARGER BUTTON TEXT
                color="black"
            )
    
    def update_display(self):
        """Update the display with current input"""
        self.canvas.delete(self.display_text)
        self.display_text = self.canvas.create_text(
            CANVAS_WIDTH - 20, DISPLAY_HEIGHT - 40,
            text=self.current_input,
            anchor="e",
            font="Arial 60 bold",  # Consistent large display font
            color=self.text_color
        )
    
    def handle_button_click(self, label):
        """Handle button click events with DEBOUNCING"""
        current_time = time.time()
        if current_time - self.last_click_time < 0.5:  # 500ms delay between clicks
            return
            
        self.last_click_time = current_time
        
        if label.isdigit():
            if self.current_input == "0" or self.reset_input_next:
                self.current_input = label
                self.reset_input_next = False
            else:
                self.current_input += label
        elif label == 'C':
            self.current_input = "0"
            self.stored_value = None
            self.current_operation = None
        elif label in ['+', '-', '*', '/']:
            if self.stored_value is not None and not self.reset_input_next:
                self.calculate_result()
            self.stored_value = float(self.current_input)
            self.current_operation = label
            self.reset_input_next = True
        elif label == '=':
            if self.stored_value is not None and self.current_operation:
                self.calculate_result()
                self.current_operation = None
        
        self.update_display()
    
    def calculate_result(self):
        """Perform the calculation"""
        try:
            current_value = float(self.current_input)
            if self.current_operation == '+':
                result = self.stored_value + current_value
            elif self.current_operation == '-':
                result = self.stored_value - current_value
            elif self.current_operation == '*':
                result = self.stored_value * current_value
            elif self.current_operation == '/':
                result = self.stored_value / current_value
            
            # Format the result
            self.current_input = str(int(result) if result.is_integer() else result)
            self.stored_value = result
            self.reset_input_next = True
        except ZeroDivisionError:
            self.current_input = "Error"
            self.stored_value = None
            self.current_operation = None
    
    def run(self):
        """Run the calculator main loop with PROPER CLICK HANDLING"""
        click_active = False
        
        while True:
            x = self.canvas.get_mouse_x()
            y = self.canvas.get_mouse_y()
            
            if x is not None and y is not None:
                if not click_active:  # New click
                    click_active = True
                    for (x1, y1, x2, y2), label in self.button_areas:
                        if x1 <= x <= x2 and y1 <= y <= y2:
                            self.handle_button_click(label)
                            break
            else:
                click_active = False  # Mouse released
                
            time.sleep(0.05)

if __name__ == "__main__":
    calculator = GraphicalCalculator()
    calculator.run()
