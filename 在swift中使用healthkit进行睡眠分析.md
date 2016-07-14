#在Swift中使用HealthKit进行睡眠分析
如今，睡眠革命成为了流行风尚，用户比以往更关注的不仅仅是了解他们睡眠的时间了，还有通过分析他们一段时间的睡眠数据来展现他们睡眠的总体趋势。随着硬件、移动等技术的发展，这一领域成为一个不断发展的有着光明前景的话题。

苹果提供了一个很酷的安全的方式与用户的个人健康信息交互，并将信息通过内置的Health app进行安全存储。你不仅可以使用HealthKit创建健康应用，还可以用来访问睡眠分析数据。

在本教程中，我将简单介绍一下HealthKit框架，并演示如何构建一个简单的睡眠分析app。

###序言
HealthKit框架提供了一个叫HealthKit存储的结构来将数据保存在加密数据库中。你可以通过`HKHealthStore`类访问这个数据库。iPhone 和 Apple Watch 都有它们独立的HealthKit存储。Health的数据会在Apple Watch和iPhone间同步；旧数据会定期从Apple Watch清理以节省空间。HealthKit 和 Health app不能在iPad上使用。

如果你想创建一个基于健康数据的iOS或watchOS应用程序，HealthKit是一个强大的工具。HealthKit可以管理多种来源的数据，并自动基于用户的偏好将不同来源的数据进行合并。应用程序还可以访问每个数据源的原始数据，并自行对数据进行合并。不光是对身体数据、健身数据和营养数据的测量，该数据也可用于睡眠分析。

接下来，我将告诉你们如何利用HealthKit框架在iOS上保存和访问睡眠分析数据，对于watchOS也同样适用。请注意，本教程是基于Swift2.0和Xcode7的，所以请确保你的Xcode版本号不低于7。

在继续之前，请先下载[起始项目](https://github.com/appcoda/SleepAnalysis/blob/master/SleepAnalysisStarter.zip?raw=true)并解压。我已经为你创建好了实现基本功能的用户界面。当你运行起始项目时，你会看到一个计时器的UI，显示你按下Start键后经过的时间。

###使用HealthKit Framework
我们这个app的目标是存储睡眠分析数据，并通过`Start`和`Stop`按键读取数据。要使用HealthKit，你需要先在你的app的bundle中对HealthKit功能进行授权。在你的工程中，导航到你的当前target->capabilities，然后打开HealthKit开关。

![image](http://www.appcoda.com/wp-content/uploads/2016/05/HealthKit-allow-1240x775.png)

接下来，你需要使用如下的代码来在`ViewController`类中创建一个`HKHealthStore`的实例：

	    let healthStore = HKHealthStore()

稍后我们将使用这个`HKHealthStore`的实例来访问HealthKit的存储。

正如上文所述，HealthKit允许用户管理他们的健康数据。所以，在你需要访问（读/写）用户的睡眠分析数据前，你需要先请求用户的许可。首先，导入内置的`HealthKit`框架，并改写`viewDidLoad`方法，代码如下：

	override func viewDidLoad() {
    	super.viewDidLoad()
    
    	let typestoRead = Set([
       HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis)!
        ])
    
    	let typestoShare = Set([
        HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis)!
        ])
    
    	self.healthStore.requestAuthorizationToShareTypes(typestoShare, readTypes: typestoRead) { (success, error) -> Void in
       		if success == false {
            NSLog(" Display not allowed")
        	}
   		}
	}
	
这段代码将提示用户允许或拒绝app所请求的权限。通过完成时的代码块，你可以通过对success或error的处理，来获得最终的结果。用户不必允许你的app所请求的所有权限，你必须在你的app里妥善地处理错误。

但是出于测试的目的，现在你必须选择"Allow"选项来授权你的app访问你的设备上的健康数据。

![image](http://www.appcoda.com/wp-content/uploads/2016/05/Health-App-Permission.png)

###写入睡眠分析数据

那么问题来了，我们如何获取睡眠分析数据？根据苹果的官方文档，每个睡眠分析样本只有一个值。为了代表用户躺在床上（In-bed）和睡着(Asleep)的不同状态，HealthKit使用两个或更多的有着重叠时间的样本，通过比较这些样本的开始和结束时间，app可以计算一些间接的统计数据：

* 用户入睡所花时间
* 用户实际睡眠时间占用户在床上的时间的百分比
* 用户醒来的次数
* 用户躺在床上（In-bed）和睡着(Asleep)的总时间

![image](http://www.appcoda.com/wp-content/uploads/2016/05/record_sleep_data.png)

简单地说，按照如下的方法来存储用户睡眠分析数据到HealthKit存储：

1. 对开始和结束的时间定义两个`NSDate`对象。
2. 然后使用`HKCategoryTypeIdentifierSleepAnalysis`创建一个`HKObjectType`的实例。
3. 创建一个`HKCategorySample`类型的新对象。通常使用分好类的样本记录睡眠数据，每个样本代表用户躺在床上（In-bed）或睡着(Asleep)的时间段。所以需要创建一个InBed和Asleep的样本，并且它们有着重叠的时间。
4. 最后，使用`HKHealthStore`中的`saveObject`保存对象。

> 编者按：对于样本的类型，你可以在[HealthKit Constants Reference](https://developer.apple.com/library/ios/documentation/HealthKit/Reference/HealthKit_Constants/index.html#//apple_ref/doc/uid/TP40014710)中查阅。

把上述步骤翻译成Swift，下面就是存储用户躺在床上（In-bed）和睡着(Asleep)时的睡眠分析数据的代码片段，请将这个方法添加到`ViewController`类中：

	func saveSleepAnalysis() {
	    
	    // alarmTime and endTime are NSDate objects
	    if let sleepType = HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis) {
	        
	        // we create our new object we want to push in Health app
	        let object = HKCategorySample(type:sleepType, value: HKCategoryValueSleepAnalysis.InBed.rawValue, startDate: self.alarmTime, endDate: self.endTime)
	        
	        // at the end, we save it
	        healthStore.saveObject(object, withCompletion: { (success, error) -> Void in
	            
	            if error != nil {
	                // something happened
	                return
	            }
	            
	            if success {
	                print("My new data was saved in HealthKit")
	                
	            } else {
	                // something happened again
	            }
	            
	        })
	        
	        
	        let object2 = HKCategorySample(type:sleepType, value: HKCategoryValueSleepAnalysis.Asleep.rawValue, startDate: self.alarmTime, endDate: self.endTime)
	        
	        healthStore.saveObject(object2, withCompletion: { (success, error) -> Void in
	            if error != nil {
	                // something happened
	                return
	            }
	            
	            if success {
	                print("My new data (2) was saved in HealthKit")
	            } else {
	                // something happened again
	            }
	            
	        })
	        
	    }
	    
	}


	
这个函数可以在我们希望存储睡眠分析数据到HealthKit时被调用。

###读取睡眠分析数据

要读取睡眠分析数据，我们需要创建一个查询。首先应该针对`HKCategoryTypeIdentifierSleepAnalyze`定义一个`HKObjectType`的类型。可能还需要使用一个谓词通过NSDate对象`startDate` 和 `endDate`来根据所需要的时间段来筛选读取到的数据。还要创建一个`sortDescriptor`对读取到的查询进行整理来选择需要的结果。

读取睡眠分析数据的代码如下：

	func retrieveSleepAnalysis() {
	    
	    // first, we define the object type we want
	    if let sleepType = HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis) {
	        
	        // Use a sortDescriptor to get the recent data first
	        let sortDescriptor = NSSortDescriptor(key: HKSampleSortIdentifierEndDate, ascending: false)
	        
	        // we create our query with a block completion to execute
	        let query = HKSampleQuery(sampleType: sleepType, predicate: nil, limit: 30, sortDescriptors: [sortDescriptor]) { (query, tmpResult, error) -> Void in
	            
	            if error != nil {
	                
	                // something happened
	                return
	                
	            }
	            
	            if let result = tmpResult {
	                
	                // do something with my data
	                for item in result {
	                    if let sample = item as? HKCategorySample {
	                        let value = (sample.value == HKCategoryValueSleepAnalysis.InBed.rawValue) ? "InBed" : "Asleep"
	                        print("Healthkit sleep: \(sample.startDate) \(sample.endDate) - value: \(value)")
	                    }
	                }
	            }
	        }
	        
	        // finally, we execute our query
	        healthStore.executeQuery(query)
	    }
	}


这段代码通过查询HealthKit获取所有的睡眠分析数据，并按降序排序。然后每条查询打印出startDate、endDate及值的类型，即InBed或Asleep。我设置了30为上限来读取最近30个样本，你也可以通过谓词的方法自定义开始和结束的时间。

###App测试

在demo程序中，我使用NSTimer显示你按下Start键后经过的时间。`NSDate`对象在点击Start和Stop按键时创建，用来将睡眠分析数据保存为经过的时间。在`stop` 的action方法中，你可以调用`saveSleepAnalysis ()`和`retriveSleepAnalisis ()`方法来保存和获取睡眠数据。

	@IBAction func stop(sender: AnyObject) {
	    endTime = NSDate()
	    saveSleepAnalysis()
	    retrieveSleepAnalysis()
	    timer.invalidate()
	}
	
	
在你的app中， 你可能需要更改NSDate对象选择相关的开始和结束时间（可能不同）来保存躺在床上（InBed）和睡着时（Asleep）的状态值。

修改完程序之后，你可以运行一下demo并开始计时，运行几分钟后点击Stop键。然后打开Health app，你会在里面发现睡眠的数据。

![image](http://www.appcoda.com/wp-content/uploads/2016/06/sleep-analysis-test-1240x878.png)

###一些对HealthKit App的建议

HealthKit能够为app开发者轻松地共享和访问用户数据提供一个通用的平台，避免数据可能存在的重复或矛盾。苹果的审核指南对于使用HealthKit的app来说很详细，请求读/写权限而不清楚地表达用途将导致app被拒绝上架。

向Health App存储假数据或错误的数据的app也会被拒绝。这意味着，你不能单纯使用算法来计算出像本教程一样的不同的健康值。你应该尝试使用内置的传感器的数据来读取和处理任何参数来避免计算出错误的数据。

点击[这里](https://github.com/appcoda/SleepAnalysis)下载完整的Xcode工程。

