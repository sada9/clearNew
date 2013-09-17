package clear.driver;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.lang.reflect.Method;
import java.net.MalformedURLException;
import java.net.URL;
import java.text.DecimalFormat;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Properties;
import java.util.concurrent.TimeUnit;
import java.util.logging.FileHandler;
import java.util.logging.Level;
import java.util.logging.LogRecord;
import java.util.logging.Logger;
import java.util.logging.SimpleFormatter;
import java.util.regex.Pattern;

import org.apache.commons.io.FileUtils;
import org.apache.commons.lang3.RandomStringUtils;
import org.apache.xerces.impl.xs.identity.Selector.Matcher;

import org.openqa.selenium.Alert;
import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.NoAlertPresentException;
import org.openqa.selenium.NoSuchElementException;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.Platform;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.firefox.FirefoxBinary;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxProfile;
import org.openqa.selenium.ie.InternetExplorerDriver;
import org.openqa.selenium.interactions.Actions;
import org.openqa.selenium.remote.Augmenter;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.remote.UnreachableBrowserException;
import org.openqa.selenium.support.ui.ExpectedCondition;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.support.ui.Wait;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.Assert;
import org.testng.ITestContext;
import org.testng.Reporter;
import org.testng.annotations.AfterClass;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.DataProvider;
import org.testng.xml.XmlTest;
import org.testng.Assert.*;

import clear.utils.ExcelData;
import clear.utils.LogFormatter;
import clear.utils.OraDBData;
import clear.utils.PropertyLoader;
import clear.utils.TestUtils;

import com.google.common.base.Function;

import test.pat.common.PATCommon;

/**
* This class base class of all the test scripts, include generic and framework methods
* @author ptt4kor
* @version 1.0
*/
@SuppressWarnings("unused")
public  class TestDriver {

	public static Properties config = null;
	public static Properties or = null;
	public static PropertyLoader propLoader = null;
	public static WebDriver dr = null;
	public static ExcelData xl = null;
	public static OraDBData db = null;
	
	public static FileInputStream fip = null;
	public static String aut; 
	public static String autPath;
	public static String orPath;
	public static String configPath;
	public static String dataSheetPath;
	public static String executionMachine;
	public static String buildNumber;
	public static String tester;
	public String actual, expected;
	static URL url = null;
	public static TestUtils testUtils = null;
	
	static FileHandler hand = null;
	static Logger log = null;
	
	public static enum LogType{ INFO, PASS, SOFTFAIL , WARNING, 
									SCREENSHOT, UNCOMPLETED, TEXTLOGONLY, HARDFAIL};
	
	
	
	/**
	 * This method is to initiate the test script execution. 
	 * @param 
	 * @return
	 * @throws 
	 * @author ptt4kor
	 */
    @BeforeClass									
	public static void initExecution(ITestContext context) {
		
		db = new OraDBData();
		propLoader = new PropertyLoader();
		
		//validate test setup 
		testSetupValidator(context);
				
		//intitate ExcelData
		xl =new ExcelData();
		
		//create testUtils object
		testUtils = new TestUtils();
	
		// load config.properties
		config = new Properties();
		try {
			fip = new FileInputStream(configPath + "/config.properties");
			config.load(fip);
			ReportLog("Load config.prop ", LogType.PASS);
			
		} catch (Exception e) {
			ReportLog("Error while loading config.properties file", LogType.UNCOMPLETED);
			e.printStackTrace();
		}
		
		//load object repository
		or = new Properties();
		or = propLoader.loadOR(new File(orPath));

		
		// create driver object based on browser
		try {
			 url = new URL( "http", executionMachine, 4444, "/wd/hub" );
		} catch (MalformedURLException e) {
			ReportLog("Execution machine ip or port or path is incorrect", LogType.UNCOMPLETED);
		}
		
		
		if (config.getProperty("BROWSER").equals("FF")) {
			ReportLog("Browser to be used for testing: Firefox ", LogType.INFO);
			/*FirefoxProfile profile = new FirefoxProfile();
			profile.setEnableNativeEvents(true);
			profile.setPreference("browser.download.folderList", 2);
			profile.setPreference("browser.download.manager.showWhenStarting",false);*/
			DesiredCapabilities ffCapabilities = DesiredCapabilities.firefox();
			dr = new  RemoteWebDriver( url, ffCapabilities);

		} else if (config.getProperty("BROWSER").equals("IE")) {
			ReportLog("Browser to be used for testing: IE ", LogType.INFO);
			System.setProperty("webdriver.ie.driver","C:/servers/ie/IEDriverServer.exe");
			
			DesiredCapabilities ieCapabilities = DesiredCapabilities.internetExplorer();
			
			ieCapabilities.setJavascriptEnabled(true);
			ieCapabilities.setBrowserName("internet explorer"); 
			ieCapabilities.setCapability(InternetExplorerDriver.INTRODUCE_FLAKINESS_BY_IGNORING_SECURITY_DOMAINS,true);
			ieCapabilities.setPlatform(Platform.ANY);
			
			//dr = new InternetExplorerDriver(ieCapabilities);
			try{
			dr = new RemoteWebDriver( url, ieCapabilities);
			}
			catch(UnreachableBrowserException e){
				ReportLog("Selenium server is NOT started or unreachable", LogType.UNCOMPLETED);
			}
			

		} else {
			ReportLog("Browser cannot be initialized - check config.prop, valid browser values FF or IE ", LogType.UNCOMPLETED);
		}
}
	
	static
	{ 
		try {
			FileHandler hand = new FileHandler("test-report/log/log" + (new SimpleDateFormat("ddMMMyyHHmma").format(new Date())) + ".log");
			
			LogFormatter slf = new LogFormatter();
			
			//hand.setFormatter(new SimpleFormatter());
			hand.setFormatter(slf);
			hand.setLevel(Level.ALL);
			log = Logger.getLogger("TEXTLOG");
			log.addHandler(hand);
		}
		 catch (Exception e) {
			 ReportLog("Failed to create log file..",LogType.SOFTFAIL);
			e.printStackTrace();
		}
	}

	/**
	 * This method is to clean up the setup once the execution is done.
	 * @param 
	 * @return
	 * @throws 
	 * @author ptt4kor
	 */
	@AfterClass
	public static void cleanSetup() {
		dr.close();
	}
	
	/**
	 * This method supplies data to test scripts
	 * @param 
	 * @return
	 * @throws 
	 * @author ptt4kor
	 */
	
	@DataProvider(name="InputDataSupplier")
	public static Object[][] dataSupplier(Method method, ITestContext context){
		
		String sheetName = null;
		Object[][] data = null;
		
		
		
		try {
			// Get the class name using method.
			String sheetNameTemp[] = method.toString().split("\\.");

			sheetName = dataSheetPath + "//"
					+ sheetNameTemp[3].toString() + ".xls";
			data = xl.GetSheetData(sheetName);
			ReportLog("Data Sheet loaded successfully" + sheetName,
					LogType.PASS);
		}

		catch (FileNotFoundException e) {
			ReportLog("Data Sheet " + sheetName + " Not found. AUT: " + aut,
					LogType.UNCOMPLETED);
			e.printStackTrace();
		}

		catch (IOException e) {
			ReportLog("IO error while loading " + sheetName + " for AUT:" + aut,
					LogType.UNCOMPLETED);
			e.printStackTrace();
		}

		catch (Exception e) {
			ReportLog("Exception caught while loading  " + sheetName
					+ "for AUT:" + aut, LogType.UNCOMPLETED);
			e.printStackTrace();
		}
		
		return data;
		}
	
	
	/**
	 * Method to validate the test setup, this will run before every test script. 
	 * @param ITestContext 
	 * @return
	 * @throws 
	 * @author ptt4kor
	 */
	public static void testSetupValidator(ITestContext context){
		
		File directory = null;
		
		ReportLog("Validating test setup..", LogType.TEXTLOGONLY);
		
		//read the aut from XML
		aut = context.getCurrentXmlTest().getParameter("aut").trim();
		executionMachine = context.getCurrentXmlTest().getParameter("ip").trim();
		buildNumber = context.getCurrentXmlTest().getParameter("build").trim();
		tester = "PTT4KOR";
		
		//check if application name is not passed
		if(aut.isEmpty()){
			ReportLog("Unable to find 'aut' parameter in the xml", LogType.UNCOMPLETED);
		}
		else{
			ReportLog("Application under test: " + aut , LogType.TEXTLOGONLY);
		}
		
		if(executionMachine.isEmpty()){
			ReportLog("Unable to find 'ip' parameter in the xml", LogType.UNCOMPLETED);
		}
		else{
			ReportLog("Execution machine: " + executionMachine , LogType.TEXTLOGONLY);
		}
		
		//check existance of app folder inside test
		autPath = "src/test/" + aut;
		directory = new File(autPath);
		if(directory.exists() && directory.isDirectory()){
			ReportLog("Folder " + autPath +  " exists", LogType.TEXTLOGONLY);
		}
		else{
			ReportLog("Folder " + autPath + " NOT exists", LogType.UNCOMPLETED);
		}
		
		//check existance of config folder
		configPath = autPath + "/config";
		directory = new File(configPath);
		if(directory.exists() && directory.isDirectory()){
			ReportLog("Config folder " + configPath +  " exists", LogType.TEXTLOGONLY);
		}
		else{
			ReportLog("Config folder " + configPath + " NOT exists", LogType.UNCOMPLETED);
		}
				
		//check existance of OR folder
		orPath = autPath + "/or";
		directory = new File(orPath);
		if(directory.exists() && directory.isDirectory()){
			ReportLog("Object Repository folder " + orPath +  " exists", LogType.TEXTLOGONLY);
		}
		else{
			ReportLog("Object Repository folder " + orPath + " NOT exists", LogType.UNCOMPLETED);
		}
		
		//check existance of data sheet folder
		dataSheetPath = "resources/datasheet/" + aut ;
		directory = new File(dataSheetPath);
		if(directory.exists() && directory.isDirectory()){
			ReportLog("Data Sheet folder " + dataSheetPath +  " exists", LogType.TEXTLOGONLY);
		}
		else{
			ReportLog("Data Sheet folder " + dataSheetPath + " NOT exists", LogType.UNCOMPLETED);
		}
		ReportLog("Test setup successful", LogType.TEXTLOGONLY);
    }
	
	/**
	 * This method is to be called to ensure object present before the action
	 * @param locator
	 * @throws  
	 * @author ptt4kor
	 * 
	 */
	
	public static boolean isElementPresent(By locator, String elementName) {
		boolean isPresent= false;
		if(waitForElementToBePresent(locator)){
				ReportLog("Element " + elementName + " present. Locator text:" + locator.toString(), LogType.TEXTLOGONLY);
				isPresent = true;
			}
			else{
				ReportLog("Element " + elementName + " NOT present. Locator text:" + locator.toString(), LogType.HARDFAIL);
			}
		return isPresent;
	}
	
	/**
	 * This method is 
	 * @param locator
	 * @throws  
	 * @author ptt4kor
	 * 
	 */
	
	public static boolean isElementVisible(By locator) {
		boolean isVisible= false;
		try{
			WebElement we = dr.findElement(locator);
			isVisible = we.isEnabled();
		}
		catch(Exception e){
			return false;
		}
		
		return isVisible;
	}
	

	
	/**
	 * This method is to be called to click buttons in the decision question
	 * @param elementInfo
	 * @throws  
	 * @author ptt4kor
	 * 
	 */
	
	public static void clickElementUsingId(String elementInfo) {
		By locator = By.id(getIdentifier(elementInfo));
		
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			 dr.findElement(locator).click();
			 ReportLog("Click on '"  + getIdentifierText(elementInfo) + "' button/link", LogType.PASS);
		}
	}
	
	/**
	 * This method is to be called to click buttons in the decision question
	 * @param elementName
	 * @throws InterruptedException 
	 * @author ptt4kor
	 * 
	 */
	public static void clickElementUsingName(String elementInfo) {
		
		By locator = By.name(getIdentifier(elementInfo));
		
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			 dr.findElement(locator).click();
			 ReportLog("Click on '"  + getIdentifierText(elementInfo) + "' button/link", LogType.PASS);
		}
				
	}
	
	/**
	 * This method is to be called to click buttons in the decision question
	 * @param xpath
	 * @throws InterruptedException 
	 * @author ptt4kor
	 * 
	 */
	public static void clickElementUsingXpath(String elementInfo){
		
		By locator = By.xpath(getIdentifier(elementInfo));
		
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
		 dr.findElement(locator).click();
		 ReportLog("Click on '"  + getIdentifierText(elementInfo) + "' button/link", LogType.PASS);
		}
   }
	
	/**
	 * This method is to be called to click link using its text
	 * @param elementInfo
	 * @throws InterruptedException 
	 * @author ptt4kor
	 * 
	 */
	public static void clickLinkUsingText(String elementInfo) {
		By locator = By.linkText(getIdentifier(elementInfo));
		
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
		 dr.findElement(locator).click();
		 ReportLog("Click on '"  + getIdentifierText(elementInfo) + "' button/link", LogType.PASS);
		}
   }
	
	
	/**
	 * This method is to be called to enter text using element id
	 * @param elementInfo, text
	 * @throws InterruptedException 
	 * @author ptt4kor
	 * 
	 */
	public static void typeTextUsingId(String elementInfo, String text) throws InterruptedException{
		By locator = By.id(getIdentifier(elementInfo));
		
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			 dr.findElement(locator).sendKeys(text);
			 ReportLog("Type text '" + text + "' into " + getIdentifierText(elementInfo) ,LogType.PASS);
		}
				
	}
	
	/**
	 * This method is to be called to enter text using element id
	 * @param elementInfo, text
	 * @throws InterruptedException 
	 * @author ptt4kor
	 * 
	 */
	public static void typeTextUsingName(String elementInfo, String text) throws InterruptedException{
		By locator = By.name(getIdentifier(elementInfo));
		
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			 dr.findElement(locator).sendKeys(text);
			 ReportLog("Type text '" + text + "' into " + getIdentifierText(elementInfo) ,LogType.PASS);
			
		}
				
	}

	
	
	
	/**
	 * This method is to wait for page to load completely til the document status is ready
	 * @param 
	 * @return
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static void waitForPageLoad() throws InterruptedException
	{
		long startTime, endTime = 0;
		double duration = 0;
		try{
			startTime = System.nanoTime();
			
			ExpectedCondition<Boolean> pageLoadCondition = new
			        ExpectedCondition<Boolean>() {
			            public Boolean apply(WebDriver driver) {
			                return ((JavascriptExecutor)driver).executeScript("return document.readyState").equals("complete");
			            }
			        };
			        
			    WebDriverWait wait = new WebDriverWait(dr, 60);
			    wait.until(pageLoadCondition);
			    
			 endTime = System.nanoTime();
			 duration = (endTime - startTime)/ 1.0E09; 
			 
			 
			 ReportLog("Wait for page load. [waited " + String.valueOf(new DecimalFormat("##.00").format(duration))  + " secs]", LogType.INFO);
			}
			catch(Exception e){
				ReportLog("Error while waiting for page load.", LogType.WARNING);
			}
	}

	/**
	 * Fluent wait. Waits for element to be present on the screen by polling every 3 secs
	 * @param locator
	 * @throws 
	 * @return boolean true if present, false if not present
	 * @author ptt4kor
	 */
	public static boolean waitForElementToBePresent(final By locator)  {
		boolean isPresent = false;
		WebElement element = null;
		try{
		 Wait<WebDriver> wait = new FluentWait<WebDriver>(dr)  
	             .withTimeout(30, TimeUnit.SECONDS)  
	             .pollingEvery(3, TimeUnit.SECONDS)  
	             .ignoring(NoSuchElementException.class); 

		 element = wait.until(new Function<WebDriver, WebElement>() {  
	           public WebElement apply(WebDriver driver) {  
	             return driver.findElement(locator);  
	            }  
	      });    
		}
		
		catch(Exception e){
			ReportLog("Wait for element timed out after 30 secs.", LogType.WARNING);
		}
		
		 return element!=null;
	}
	
	/**
	 * Waits in secods
	 * @param seconds
	 * @throws 
	 * @return 
	 * @author ptt4kor
	 */
	public static void wait(final int secs)  {
		
		try{
			Thread.sleep(secs * 1000);
		}
		
		catch(Exception e){
			ReportLog("Exception in wait", LogType.WARNING);
		}
		
	}
	
	
	/**
	 * This method is to check whether alert box is present or not
	 * @param action on alert eg. accept, dismiss, ok
	 * @return
	 * @throws 
	 * @author ptt4kor
	 */
	
	public static boolean isAlertPresent(String action) {
		  boolean presentFlag = false;
		 
		  try {
		 
		   // Check the presence of alert
		   Alert alert = TestDriver.dr.switchTo().alert();
		   // Alert present; set the flag
		   presentFlag = true;
		   
		   // if present consume the alert
		  if(action.equalsIgnoreCase("accept")){			
		   alert.accept();
		  }
		  else{
			  alert.dismiss();
		  }
		  } catch (NoAlertPresentException ex) {
		   // Alert not present
		   //ex.printStackTrace();
		  }
		  
		  return presentFlag;
		 
		 }	
	
	
	/**
	 * This method reads the inner text
	 * @param xpath of web object
	 * @return innerText of xpath element
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static String getInnerTextUsingXpath(String elementInfo) throws InterruptedException
	{
		By locator = By.xpath(getIdentifier(elementInfo));
		
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			return dr.findElement(locator).getText();
			}
		else{
			ReportLog("getText Failed for " + getIdentifierText(elementInfo),LogType.HARDFAIL);
			return  "NULL";
		}
		
	}
	
	/**
	 * This method reads property values of an element
	 * @param xpath of web object, property for which value to be read
	 * @return property value if exist otherwise null
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static String getElementPropertyUsingXpath(String elementInfo, String property) throws InterruptedException
	{
		By locator = By.xpath(getIdentifier(elementInfo));
		
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			return dr.findElement(locator).getAttribute(property);
			}
		else{
			ReportLog("Unable to read " + property + " property of " + getIdentifierText(elementInfo) + "element" ,LogType.HARDFAIL);
			return  "NULL";
		}
		
	}
	
	/**
	 * This method gets row count in table
	 * @param xpath of the table
	 * @return row count in the table
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static int getTableRowCountUsingXpath(String elementInfo) throws InterruptedException
	{
		
		By locator = By.xpath(getIdentifier(elementInfo)+ "//tr");
		int rowCount = 0;
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			 List<WebElement> trs = dr.findElements(locator);
		     rowCount = trs.size();
		    }
		else{
			ReportLog("Unable to get the row count of " + getIdentifierText(elementInfo) + "element" ,LogType.HARDFAIL);
		}
		
		return rowCount;
	}
	
	/**
	 * This method gets columns count in table(first row)
	 * @param xpath of the table
	 * @return row count in the table
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static int getTableColCountUsingXpath(String elementInfo) throws InterruptedException
	{
		
		By locator = By.xpath(getIdentifier(elementInfo)+ "//tr[1]/td");
		int colCount = 0;
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			 List<WebElement> trs = dr.findElements(locator);
			 colCount = trs.size();
		    }
		else{
			ReportLog("Unable to get the col count of " + getIdentifierText(elementInfo) + "element" ,LogType.HARDFAIL);
		}
		
		return colCount;
	}
	
	/**
	 * This method returns the tag info
	 * @param xpath of the td or div which contains UL tags
	 * @return row count in the table
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static int getUlTagCount(String elementInfo) throws InterruptedException
	{
		
		By locator = By.xpath(getIdentifier(elementInfo)+ "/ul");
		int colCount = 0;
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			 List<WebElement> trs = dr.findElements(locator);
			 colCount = trs.size();
		    }
		else{
			ReportLog("Unable to get the Ul tag count of " + getIdentifierText(elementInfo) + "element" ,LogType.HARDFAIL);
		}
		
		return colCount;
	}
	
	/**
	 * This method returns the tag info
	 * @param xpath of the td or div which contains UL tags
	 * @return row count in the table
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static int getDivTagCount(String elementInfo) throws InterruptedException
	{
		
		By locator = By.xpath(getIdentifier(elementInfo)+ "/div");
		int colCount = 0;
		if(isElementPresent(locator, getIdentifierText(elementInfo))){
			 List<WebElement> trs = dr.findElements(locator);
			 colCount = trs.size();
		    }
		else{
			ReportLog("Unable to get the div tag count of " + getIdentifierText(elementInfo) + "element" ,LogType.HARDFAIL);
		}
		
		return colCount;
	}
	
	/**
	 * This method is to get the identifier from Object repository property file. (value before the #)
	 * @param value
	 * @return 
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static String getIdentifier(String value) 
	{
		String[] tempSplit = value.toString().split("#");
		return tempSplit[0].trim();
	}
	
	/**
	 * This method is to get the identifier info from Object repository property file. (value after the #)
	 * @param value
	 * @return 
	 * @throws Exception
	 * @author ptt4kor
	 */
	
	public static String getIdentifierText(String value) 
	{
		String[] tempSplit = value.toString().split("#");
		if(tempSplit.length > 1){
			return tempSplit[1].trim();
		} else
		{
			return tempSplit[0].trim();
		}
			
	}
	
	
	/**
	 * This method logs the message or image to the PDF report 
	 * @param message to be printed on report, LogType 
	 * @return type of log could be any one of LogType
	 * @throws 
	 * @author ptt4kor
	 */
	public static void ReportLog(String message, LogType type){
		//get the name of the method
		
		String methodName =Thread.currentThread().getStackTrace()[2].getMethodName();
	switch(type){
		case INFO:
			Reporter.log(message+ " [I]");
			log.info("["+ methodName + "] - [INFO] - " + message );
			break;
		case PASS:
			Reporter.log(message+ " [P]");
			log.info("["+ methodName + "] - [PASS] - " + message );
			break;
		case SOFTFAIL:
			Reporter.log(message+ " [F]");
			log.info("["+ methodName + "] - [SOFTFAIL] - " + message );
			takeSaveScreenShot();
			break;
		case WARNING:
			Reporter.log(message+ " [W]");
			log.info("["+ methodName + "] - [WARNING] - " + message );
			break;
		case SCREENSHOT:
			Reporter.log(message+ " [J]");
			log.info("["+ methodName + "] - [JPG] - " + "Screenshot taken, file name is " + message + ".jpg" );
			break;
		case UNCOMPLETED:
			Reporter.log(message+ " [U]");
			log.info("["+ methodName + "] - [UNCOMPLETED] - " + message );
			Assert.fail(message);
			break;
		case TEXTLOGONLY:
			log.info("["+ methodName + "] - [LOG] - " + message );
			break;
		case HARDFAIL:
			Reporter.log(message+ " [S]");
			log.info("["+ methodName + "] - [HARDFAIL] - " + message );
			takeSaveScreenShot();
			Assert.fail(message);
			break;
		}
		
	}
	
	/**
	 * Method to take screenshot and save in location REPOER 
	 * @param message to be printed on report, LogType 
	 * @return type of log could be any one of LogType
	 * @throws 
	 * @author ptt4kor
	 */
	public static void takeSaveScreenShot(){
		String fileName = RandomStringUtils.randomAlphanumeric(4);
		//convert to augumenter, as this is TakeScreenshot is also augumentor
		WebDriver augmentedDriver = new Augmenter().augment(dr);
		
        try {
        	//without augumentor, not using remote web driver
        	//File scrFile = ((TakesScreenshot)dr).getScreenshotAs(OutputType.FILE);
        	
        	File scrFile = ((TakesScreenshot)augmentedDriver).getScreenshotAs(OutputType.FILE);
        	
			FileUtils.copyFile(scrFile, new File(config.getProperty("IMAGES_SCREENSHOTS") + fileName.trim() + ".jpg"));
			
			ReportLog(fileName, LogType.SCREENSHOT);
		
        } catch (Exception e) {
        	ReportLog("TakeScreenShot failed!", LogType.INFO);
        	ReportLog("TakeScreenShot failed." + e.getMessage(), LogType.TEXTLOGONLY);
		}

    }
	
	
	
	/**
	 * Method to compare String type values 
	 * @param message to be printed on report, LogType 
	 * @return type of log could be any one of LogType
	 * @throws 
	 * @author ptt4kor
	 */
	public static void match(String expected, String actual, String field){
		actual = actual.trim();
		expected = expected.trim();
		
		//try to match without ignoring case
		if(!actual.equals(expected)){
			if(actual.equalsIgnoreCase(expected)){
				ReportLog(field + " values matched"  , LogType.PASS);
				ReportLog(field + " values matched by ignoring case"  , LogType.WARNING);
			}
			else{
				ReportLog(field + " values NOT matched", LogType.SOFTFAIL);
			}
		}
		else{
			ReportLog(field + " values matched."  , LogType.PASS);
		}
		
		//display expected n actual
		//display in 2 lines if charcter are more
		if(expected.length()+actual.length() > 52){
		ReportLog("Expected: " + expected  , LogType.INFO);
		ReportLog("Actual: " + actual , LogType.INFO);
		}
		else{
			ReportLog("Expected: " + expected + " Actual: " + actual , LogType.INFO);
		}
			
    }
	/**
	 * Method to compare String type values 
	 * @param message to be printed on report, LogType 
	 * @return type of log could be any one of LogType
	 * @throws 
	 * @author ptt4kor
	 */
	public static void match(double expected, double actual, String field){
		if(actual == expected){
			ReportLog(field + " values matched"  , LogType.PASS);
			
		}
		else{
			ReportLog(field + " values NOT matched", LogType.SOFTFAIL);
		}
		ReportLog("Expected: " + expected + " Actual: " + actual , LogType.INFO);
    }
	
	/**
	 * Method to compare integer type values 
	 * @param message to be printed on report, LogType 
	 * @return type of log could be any one of LogType
	 * @throws 
	 * @author ptt4kor
	 */
	public static void match(int expected, int actual, String field){
		if(actual == expected){
			ReportLog(field + " values matched"  , LogType.PASS);
		}
		else{
			ReportLog(field + " values NOT matched", LogType.SOFTFAIL);
		}
		ReportLog("Expected: " + expected + " Actual: " + actual , LogType.INFO);
    }
	
	/**
	 * Method to compare String type values 
	 * @param message to be printed on report, LogType 
	 * @return type of log could be any one of LogType
	 * @throws 
	 * @author ptt4kor
	 */
	public static void match(Date expected, Date actual, String field){
		if(actual.equals(expected)){
			ReportLog(field + " values matched"  , LogType.PASS);
		}
		else{
			ReportLog(field + " values NOT matched", LogType.SOFTFAIL);
		}
		
		ReportLog("Expected: " + testUtils.getFormattedDate(expected, "MM/dd/yyyy") + " Actual: " + testUtils.getFormattedDate(actual, "MM/dd/yyyy") , LogType.INFO);
	}
	
	/**
	 * Method to hover element 
	 * @param locator 
	 * @return 
	 * @throws 
	 * @author ptt4kor
	 */
	public static void hover(By locator){
		WebElement we = dr.findElement(locator);
		Actions builder = new Actions(dr); 
		Actions hoverOver= builder.moveToElement(we);
		hoverOver.perform();
	}
	
}
