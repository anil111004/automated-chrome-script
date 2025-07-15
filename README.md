# automated-chrome-script
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

def create_stealth_driver():
    # Configure Chrome options for stealth and functionality
    chrome_options = Options()
    
    # Basic stealth options
    chrome_options.add_argument("--disable-blink-features=AutomationControlled")
    chrome_options.add_argument("--disable-infobars")
    chrome_options.add_argument("--disable-notifications")
    chrome_options.add_argument("--disable-popup-blocking")
    chrome_options.add_argument("--disable-extensions")
    
    # Disable automation flags
    chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
    chrome_options.add_experimental_option('useAutomationExtension', False)
    
    # Enable JavaScript and disable web security (for copy-paste functionality)
    chrome_options.add_argument("--disable-web-security")
    chrome_options.add_argument("--allow-running-insecure-content")
    
    # Headless mode can be enabled if needed (but may trigger bot detection)
    # chrome_options.add_argument("--headless")
    
    # Create driver with options
    driver = webdriver.Chrome(options=chrome_options)
    
    # Execute CDP commands to further prevent detection
    driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
        "source": """
            Object.defineProperty(navigator, 'webdriver', {
                get: () => undefined
            });
            Object.defineProperty(navigator, 'plugins', {
                get: () => [1, 2, 3, 4, 5]
            });
            Object.defineProperty(navigator, 'languages', {
                get: () => ['en-US', 'en']
            });
            window.chrome = {
                app: {
                    isInstalled: false,
                },
                webstore: {
                    onInstallStageChanged: {},
                    onDownloadProgress: {},
                },
                runtime: {
                    PlatformOs: {
                        MAC: 'mac',
                        WIN: 'win',
                        ANDROID: 'android',
                        CROS: 'cros',
                        LINUX: 'linux',
                        OPENBSD: 'openbsd',
                    },
                    PlatformArch: {
                        ARM: 'arm',
                        X86_32: 'x86-32',
                        X86_64: 'x86-64',
                    },
                    PlatformNaclArch: {
                        ARM: 'arm',
                        X86_32: 'x86-32',
                        X86_64: 'x86-64',
                    },
                    RequestUpdateCheckStatus: {
                        THROTTLED: 'throttled',
                        NO_UPDATE: 'no_update',
                        UPDATE_AVAILABLE: 'update_available',
                    },
                    OnInstalledReason: {
                        INSTALL: 'install',
                        UPDATE: 'update',
                        CHROME_UPDATE: 'chrome_update',
                        SHARED_MODULE_UPDATE: 'shared_module_update',
                    },
                    OnRestartRequiredReason: {
                        APP_UPDATE: 'app_update',
                        OS_UPDATE: 'os_update',
                        PERIODIC: 'periodic',
                    },
                },
            };
        """
    })
    
    return driver

def bypass_right_click_block(driver, element):
    """Bypass right-click restrictions using ActionChains"""
    action = ActionChains(driver)
    action.context_click(element).perform()

def bypass_copy_paste_restrictions(driver, element, text_to_paste=None):
    """Bypass copy-paste restrictions using JavaScript"""
    if text_to_paste:
        # For pasting
        driver.execute_script(f"""
            var element = arguments[0];
            var text = `{text_to_paste}`;
            element.value = text;
            element.dispatchEvent(new Event('change'));
        """, element)
    else:
        # For copying
        return driver.execute_script("""
            var element = arguments[0];
            return element.value;
        """, element)

def answer_assessment_questions(driver):
    """Automatically answer questions in an online assessment"""
    # Wait for questions to load
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.CSS_SELECTOR, ".question, [class*='question'], .quiz, [class*='quiz']"))
    )
    
    # Find all questions on the page
    questions = driver.find_elements(By.CSS_SELECTOR, ".question, [class*='question'], .quiz-question, [class*='quiz-question']")
    
    for i, question in enumerate(questions, 1):
        try:
            print(f"Processing question {i}")
            
            # Try to find radio buttons (multiple choice)
            radio_options = question.find_elements(By.CSS_SELECTOR, "input[type='radio']")
            if radio_options:
                # Select the first option (modify as needed)
                radio_options[0].click()
                print(f"Selected option 1 for question {i}")
                continue
            
            # Try to find checkboxes
            checkboxes = question.find_elements(By.CSS_SELECTOR, "input[type='checkbox']")
            if checkboxes:
                # Check the first box (modify as needed)
                checkboxes[0].click()
                print(f"Checked box 1 for question {i}")
                continue
            
            # Try to find text input
            text_input = question.find_element(By.CSS_SELECTOR, "input[type='text'], textarea")
            if text_input:
                answer = "Sample answer"  # Replace with your answer or logic
                bypass_copy_paste_restrictions(driver, text_input, answer)
                print(f"Entered text answer for question {i}")
                continue
                
        except Exception as e:
            print(f"Could not answer question {i}: {str(e)}")
            continue

def switch_tabs(driver, tab_index):
    """Switch between browser tabs"""
    windows = driver.window_handles
    if tab_index < len(windows):
        driver.switch_to.window(windows[tab_index])
        return True
    return False

def main():
    # Create stealth driver
    driver = create_stealth_driver()
    
    try:
        # Open initial page
        driver.get("https://www.google.com")
        print("Opened initial page")
        
        # Example of opening new tabs
        driver.execute_script("window.open('https://www.example.com', '_blank');")
        print("Opened new tab")
        
        # Switch between tabs
        switch_tabs(driver, 0)
        print("Switched to first tab")
        time.sleep(2)
        
        switch_tabs(driver, 1)
        print("Switched to second tab")
        time.sleep(2)
        
        # Example of answering assessment questions (replace URL with your assessment)
        driver.get("https://example-assessment.com")
        answer_assessment_questions(driver)
        
        # Example of bypassing copy-paste restrictions
        element = driver.find_element(By.CSS_SELECTOR, "input[type='text']")
        bypass_copy_paste_restrictions(driver, element, "This text was pasted programmatically")
        
        # Example of bypassing right-click restrictions
        bypass_right_click_block(driver, element)
        
    finally:
        # Keep browser open for inspection (remove this in production)
        input("Press Enter to close the browser...")
        driver.quit()

if __name__ == "__main__":
    main()
