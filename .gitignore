//

//  PromoteApp.m

//  InstaObserver

//

//  Created by Muthana on 11/17/13.

//

//



#import "PromoteApp.h"

#import "MyGlobals.h"

#import "SFHFKeychainUtils.h"



@implementation PromoteApp



@synthesize numOfNoThanks, numOfMaybeLater, predictedDate, messageId;



#define noThanksDaysIncrement 2 //each one choice of this add this number

#define maybeLaterDaysIncrement 1 //each one choice of this add this number

#define maxWaitDays 6 //Should not exceed this wait period even if the derived wait is more. e.x. 4*2 = 8/2 = 4 hits

#define noThanksLockCap 10 * noThanksDaysIncrement //how many times of this count until you lock the Promotion.

#define maybeLaterLockCap 25 * maybeLaterDaysIncrement///how many times of this count until you lock the Promotion

#define oneDayInSeconds 86400

#define oneHourInSeconds 3600


//Keychain stuff

#define promotionUserName @"not discloused"  

#define promotionServiceName @"not discloused"  

#define myPromotionPassword @"not discloused"  



//Alterating Promotion messages

#define promotionMessageHeader1 @"More 4 Free!"

#define promotionMessageText1 @"InstaFollower app has new addtional cool features. Want to downlaod it for Free?"

#define promotionMessageHeader2 @"InstaFollower!"

#define promotionMessageText2 @"InstaFollower allows you to know your most popular, engaging, liked posts and more! Do you Want to try it now?"

#define promotionMessageHeader3 @"Discover more?"

#define promotionMessageText3 @"Do you want to discover other important insights about your Instagram account using our free app InstaFollower?"

#define promotionMessageHeader4 @"More Info!" //SPECIAL MESSAGE WITH ONLY 2 OPTIONS *****

#define promotionMessageText4 @"Do you know that we have unlocked all features of InstaFollower? Do you want to try the app now?"

#define promotionMessageCount 4 // <--- Changable



#define IS_IPHONE (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPhone)

#define IS_IPHONE_5 (IS_IPHONE && [[UIScreen mainScreen] bounds].size.height == 568.0f)




- (BOOL) isActivePromotion{

    NSError *error = nil;

    NSString *password = [SFHFKeychainUtils getPasswordForUsername:promotionUserName andServiceName:promotionServiceName error:&error];

    

    if ([password isEqualToString:myPromotionPassword])

        return NO;

    else

        return YES;

    

    return NO; //default value

    

}



- (void) disablePromotion{

  

    NSError *error = nil;

    [SFHFKeychainUtils storeUsername:promotionUserName andPassword:myPromotionPassword forServiceName:promotionServiceName updateExisting:YES error:&error]; //add to stop from now on

    

}





-(NSString *) incrementCountOf:(NSString*)currElemCount byValue:(int)offset{

    

    NSString* result;

    

    result = [NSString localizedStringWithFormat:@"%i", [currElemCount intValue] + offset];

    

   // NSLog(@"Promotion - Curr Reponse New Value: %@",result );

    

    return result;

}





- (void) readStoredPromotionData{

    

    NSArray *myPath= NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);

    NSString *plistDirectory = [myPath objectAtIndex:0];

    NSString *myFile = [plistDirectory stringByAppendingPathComponent:@"PromotionLogic.plist"];

    NSMutableDictionary *plistDict  = [[NSMutableDictionary alloc] initWithContentsOfFile:myFile];

    

    NSMutableDictionary *promotionDict = [[NSMutableDictionary alloc] init]; //imporatant to intialize

    

    promotionDict = [plistDict objectForKey:@"promotionDict"];

   

    numOfNoThanks = [promotionDict valueForKey:@"numOfNoThanks"];

    numOfMaybeLater = [promotionDict valueForKey:@"numOfMaybeLater"];

    messageId =  [promotionDict valueForKey:@"messageId"];

    predictedDate = [promotionDict valueForKey:@"predictedDate"];

    

   // NSLog(@"in Read: promotionDict is : %@ ",promotionDict);

    

}



- (void) storePromotionData{

    

    NSArray *myPath= NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);

    NSString *plistDirectory = [myPath objectAtIndex:0];

    NSString *myFile = [plistDirectory stringByAppendingPathComponent:@"PromotionLogic.plist"];

    NSMutableDictionary* plistDict = [[NSMutableDictionary alloc] init];

    

    //read current values to dict to store the whole dicts

    NSMutableDictionary *promotionDict = [[NSMutableDictionary alloc] init]; //imporatant to intialize

    [promotionDict setValue:numOfNoThanks forKey:@"numOfNoThanks"];

    [promotionDict setValue:numOfMaybeLater forKey:@"numOfMaybeLater"];

    [promotionDict setValue:messageId forKey:@"messageId"];

    [promotionDict setValue:predictedDate forKey:@"predictedDate"];

    
    [plistDict  setObject:promotionDict forKey:@"promotionDict"];


    [plistDict writeToFile:myFile atomically:YES];

    

   //NSLog(@"in Store: promotionDict is : %@ ",promotionDict);

    

}


- (void) processPromotion{

    

    // Reset Manually 

    //NSError *error = nil; [SFHFKeychainUtils deleteItemForUsername:promotionUserName andServiceName:promotionServiceName error:&error];

    if (! [self isActivePromotion]){

        [[NSNotificationCenter defaultCenter] removeObserver:self];// No need to keep listening anymore at this point

        return; //stop

    }

  
    //read stored data if any
    [self readStoredPromotionData];

    

    if (!numOfNoThanks || !numOfMaybeLater || !predictedDate || !messageId){ //first time [was using && which could break]

        numOfNoThanks = @"0";

        numOfMaybeLater = @"0";

        messageId = @"1";

        //get today's datetimestamp and add a 1,2,... hours to i
        predictedDate = [self getCurrUnixTimeStampWithOffset:oneHourInSeconds*4]; //wait 4 hours to start promoting the other app. The user then is happy, trusting and may have intention to get more apps

       // NSLog(@"Promotion - in Main first predictedDate is : %@ ",predictedDate);

        [self storePromotionData]; //store and leave

    }else{

        if ([numOfMaybeLater intValue] > maybeLaterLockCap || [numOfNoThanks intValue] > noThanksLockCap){ //stop Promotion tracking

            [self disablePromotion]; // The user has had enough messages and no luck

        } else{ //check if we passed stored date

            if ([self nowPassedStoredUnixTimestamp] && !waitingPromoMsgRespns){//show Promotion message

           
                //Check for the special message

                if ([messageId isEqualToString:@"4"]){ //SHOW ONLY 2 Choices 

                    UIAlertView *promotionMessage = [[UIAlertView alloc] initWithTitle:[self getMessageHeader]

                                                                               message:[self getMessageText]

                                                                              delegate:self

                                                                     cancelButtonTitle:nil

                                                                     otherButtonTitles:@"Yes",

                                                                                       @"No",

                                                                                       nil];

                    promotionMessage.tag = 33;

                    [promotionMessage show];

                    

                } else{ //Show 4 options

                    UIAlertView *promotionMessage = [[UIAlertView alloc] initWithTitle:[self getMessageHeader]

                                                                            message:[self getMessageText]

                                                                           delegate:self

                                                                  cancelButtonTitle:nil

                                                                  otherButtonTitles:@"Yes",

                                                                                    @"I have it",

                                                                                    @"Maybe later", 

                                                                                    @"No thanks", 

                                                                                     nil];

                    promotionMessage.tag = 1;

                    [promotionMessage show];

                

                }

                

                waitingPromoMsgRespns = YES;

                

            } else{ //could delete this later

                NSLog(@"@@@@ Still Waiting To Promote the APP @@@@");

            }

        }

        

    }

    

}



- (BOOL) nowPassedStoredUnixTimestamp{

	//Testing portion

   // NSLog(@"Promotion Compare - Stored TimeStamp: %f",[predictedDate doubleValue]);

   // NSLog(@"Promotion Compare - CurrentTimeStamp: %f",[ [self getCurrUnixTimeStampWithOffset:0] doubleValue]);

    //NSLog(@"Promotion Compare - DIFF: %f",[ [self getCurrUnixTimeStampWithOffset:0] doubleValue] - [predictedDate doubleValue]); //- value means still not reached


    if (predictedDate){ //if it has value

        if ( [[self getCurrUnixTimeStampWithOffset:0] doubleValue] >  [predictedDate doubleValue] )// -> ** doubleValue  was a bug forgetting it. Commas were there too **

            return YES;

        else

            return NO;

    }

    

    return NO;

    

}



// return a string with current date with offset. (0) works
- (NSString*) getCurrUnixTimeStampWithOffset:(int)offSet{

    NSDate *date = [[NSDate alloc] initWithTimeIntervalSinceNow:offSet]; //add offset in seconds to promotion after it

    NSTimeInterval timeStampUnix = [date timeIntervalSince1970]; //convert to unix time stamp

    NSString *convertedDoubleToStr = [NSString localizedStringWithFormat:@"%f",timeStampUnix];

    

    return [self cleanTimeStampFromCommas:convertedDoubleToStr]; //commas were making the double treated as something else

}



- (NSString*) cleanTimeStampFromCommas:(NSString*)myDblAsStr {

    

    NSString *formattedTimeStamp = [myDblAsStr stringByReplacingOccurrencesOfString:@"," withString:@""];

    

    return  formattedTimeStamp;

}



- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex

{

    BOOL yesSelected = NO;//used to make one code base with handling a special case

    BOOL noSelected = NO; //used to make one code base with handling a special case

    



    // promotion message 4 options

    if (alertView.tag == 1){

        

        waitingPromoMsgRespns = NO;

        

        switch (buttonIndex) {

            case 0:

                yesSelected = YES;

                break;

                

            case 1:

                NSLog(@"I have it is selected");

                [self disablePromotion];

                [self sendPromotionTrackerToServer:[self getPromotionCommaSepStringWithAnswer:@"Have_it"]]; // Send Promotion info Have it

                break;

                

            case 2: 

                NSLog(@"Maybe later was selected");

                numOfMaybeLater = [self incrementCountOf:numOfMaybeLater byValue:maybeLaterDaysIncrement];

                messageId = [self incrementMessageIdBy1:messageId];

                [self updatePredictedDate];

                [self storePromotionData];

                break;

                

            case 3: 

                noSelected = YES;

                break;

                

            default:

                break;

        }

    }

    

    // SPECIAL MESSAGE (2 OPTIONS ONLY)

    if (alertView.tag == 33){

        

        waitingPromoMsgRespns = NO;

        

        switch (buttonIndex) {

            case 0:

                yesSelected = YES;

                break;

                

            case 1:

                noSelected = YES;

                break;

                

            default:

                break;

        }

    }

    

    //Shared to make one change if any 

    if (yesSelected){

        NSLog(@"Yes is selected");

        [self disablePromotion];

        [self sendPromotionTrackerToServer:[self getPromotionCommaSepStringWithAnswer:@"Yes"]]; // Send Promo info Yes

        [self gotToAppStore];

        

    }

    //Shared to make one change if any

    if (noSelected) {

        NSLog(@"No was selected");

        numOfNoThanks =[self incrementCountOf:numOfNoThanks byValue:noThanksDaysIncrement]; 

        messageId = [self incrementMessageIdBy1:messageId];

        [self updatePredictedDate];

        [self storePromotionData];

        

    }



}



- (void)messageComposeViewController:(MFMessageComposeViewController *)controller didFinishWithResult:(MessageComposeResult)result {

    [self dismissModalViewControllerAnimated:YES];

}



- (NSString*) getPromotionCommaSepStringWithAnswer:(NSString*)answer{
 
    NSMutableString *longStr = [NSMutableString string]; //initial value

    UIDevice *dev = [UIDevice currentDevice];

    NSString *deviceInfo = [NSString stringWithFormat:@"%@ %@", dev.model, dev.systemVersion];


	
    [longStr appendString:currAppNameForTracking]; //currAppNameForTracking

    [longStr appendString:[NSString localizedStringWithFormat:@",%@", currAppVersion]]; //App Version

    [longStr appendString:[NSString localizedStringWithFormat:@",%@", deviceInfo]]; //device info such as iPhone 5 6.1

    [longStr appendString:[NSString localizedStringWithFormat:@",%@", numOfMaybeLater]]; //mayber later count

    [longStr appendString:[NSString localizedStringWithFormat:@",%@", numOfNoThanks]]; //no thanks count

    [longStr appendString:[NSString localizedStringWithFormat:@",%@", messageId]]; //message Id that invoked response

    [longStr appendString:[NSString localizedStringWithFormat:@",%@", @"InstaFollowerFree_Promotion"]]; //Promoted App

    [longStr appendString:[NSString localizedStringWithFormat:@",%@", answer]]; //Yes or Have it

    
    return longStr;

}



- (void) sendPromotionTrackerToServer:(NSString*)promotionStrInfo{

  

    @try {

        NSString *urlString = @"http://myURL...php";

        NSURL *aUrl = [NSURL URLWithString:urlString];

        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:aUrl cachePolicy:NSURLRequestUseProtocolCachePolicy timeoutInterval:30.0]; //Give it this much. It wont delay here.

        [request setHTTPMethod:@"POST"];

        NSString *postString = [NSString localizedStringWithFormat:@"response=%@",promotionStrInfo];

        [request setHTTPBody:[postString dataUsingEncoding:NSUTF8StringEncoding]];

        

        [NSURLConnection sendAsynchronousRequest:request

                                           queue:[NSOperationQueue mainQueue]

                               completionHandler:^(NSURLResponse *response,NSData *data,NSError *error)

         {

             NSLog(@".. User Promotion Data Successfully!");

             

             

         }];

    }

    @catch (NSException *exception) {

        NSLog(@"Error in s.. Promo info");

    }

}



- (NSString*) getMessageHeader{

    

    NSString *Header = @"InstaFollower!"; //default

    

    switch ([messageId intValue]) {

        case 1:

            Header = promotionMessageHeader1;

            break;

            

        case 2:

            Header = promotionMessageHeader2;

            break;

            

        case 3:

            Header = promotionMessageHeader3;

            break;

            

        case 4:

            Header = promotionMessageHeader4;

            break;

            

        default:

            break;

    }

    

    return Header;

}



- (NSString*) getMessageText{

    

    NSString *Text = @"Our advanced Instagram tracking app has new features revealed for the first time and beautiful design. Do you want to get it now for free?"; //default

    

    switch ([messageId intValue]) {

        case 1:

            Text = promotionMessageText1;

            break;

            

        case 2:

            Text = promotionMessageText2;

            break;

            

        case 3:

            Text = promotionMessageText3;

            break;

        

        case 4:

            Text = promotionMessageText4;

            break;

            

        default:

            break;

    }

    

    return Text;

}



-(NSString *) incrementMessageIdBy1:(NSString*)currElemCount{

    

    //Resart from 1 if passed message count we have

    NSInteger messageNextCount;

    messageNextCount = [currElemCount intValue] + 1; //ALWAYS 1

    if ( messageNextCount > promotionMessageCount)

        messageNextCount = 1; //restart from first message

    NSString* result;

    result = [NSString localizedStringWithFormat:@"%i", messageNextCount ];

    

    //NSLog(@"Next Promotion Message Value: %@",result );

    

    return result;

    

}



- (void) updatePredictedDate{

    //Used to update based on current values for all choices

    int countsTotal = 1; //intial value

    if (numOfNoThanks && numOfMaybeLater){ //check if null strings. Should not be but in case

        countsTotal = [numOfNoThanks intValue] + [numOfMaybeLater intValue];

    }

    

    // should not be more than the cap (ex. 7 days)

    if (countsTotal > maxWaitDays )

        countsTotal = maxWaitDays;

    

    //NSLog(@"Promotion predictedDate Old: %@",predictedDate);

    predictedDate = [self getCurrUnixTimeStampWithOffset:oneDayInSeconds*countsTotal];

    //NSLog(@"Promotion predictedDate New: %@",predictedDate);

    

}



- (void) gotToAppStore{

    

    NSURL *URL = [NSURL URLWithString:InstaFollowerFreeURL]; //Changable

    if ([[UIApplication sharedApplication] canOpenURL:URL]) {

        [[UIApplication sharedApplication] openURL:URL];

    } else{

        

        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Cannot Access App Store"

                                                        message:@"Please try again later."

                                                       delegate:nil

                                              cancelButtonTitle:@"OK"

                                              otherButtonTitles:nil];

        [alert show];

    }

}





- (id) init

{

    self = [super init];

    if (!self) return nil;

    

  
	//subscribe to observe a system event
    [[NSNotificationCenter defaultCenter] addObserver:self

                                             selector:@selector(processPromotion)

                                                 name:UIApplicationDidBecomeActiveNotification object:nil];

    

    

    return self;

}



@end

