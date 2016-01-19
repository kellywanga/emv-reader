# emv-reader

A small program that reads and prints all the openly accessible information on an EMV card (e.g., Visa and Mastercard). This project is based on the EMV 4.3 specification from [EMVCo](http://www.emvco.com/), which can be found [here](http://www.emvco.com/specifications.aspx?id=223).

	Card detection and reset
Card detection and reset needs to be performed by the card interface functions specific to the hardware device being used. When a card is reset, it will respond with an Answer To Reset (ATR) that specifies how the terminal must interface with the card.
	Candidate list creation.
The terminal has a list containing the Application Identifier (AID) of every EMV application that it is configured to support, and the terminal must generate a candidate list of applications that are supported by both the terminal and card. The terminal may attempt to obtain a directory listing of all card applications from the card's PSE. If this is not supported or fails to find a match, the terminal must iterate through its list asking the card whether it supports each individual AID.

If there are multiple applications in the completed candidate list, or the application requires it, then the cardholder will be asked to choose an application; otherwise it may be automatically selected.
	App selection
When the application to use has been chosen, the terminal must select the application on the card, so that the card can supply the correct data records for the transaction.
	Read app data
Once the application has been selected, the terminal provides the card with any data that it requests in the PDOL and gets the processing options. The card will supply theApplication File Locator (AFL), which is used by the terminal to read the application data records from the card. These records contain the card PAN and expiry date, plus many other tags of information that are used for transaction processing such as cardholder verification and card authentication.

the Processing Options Data Object List (PDOL) used with the GET PROCESSING OPTIONS command The PDOL is a list of tags and lengths of terminal-resident data elements needed by the ICC in processing the GET PROCESSING OPTIONS command. Only data elements having the terminal as the source of the data may be referenced in the PDOL. 
If the PDOL does not exist, the GET PROCESSING OPTIONS command uses a command data field of '8300', indicating that the length of the value field in the command data is zero. 
If the PDOL exists, the terminal extracts the PDOL from the FCI of the ADF and uses it to create a concatenated list of data elements without tags or lengths

	Data Authentication
There are three types of offline Data Authentication that can be performed, but the method to be used depends on the capabilities of the card and terminal. Online-only terminals are not required to support data authentication, but all other terminals must support both SDA and DDA and may also support CDA. SDA - Static Data Authentication of the card data (e.g. account number and expiry date) to verify that it has not been modified. DDA - Dynamic Data Authentication of card and terminal data to verify that the card application and data are genuine. CDA - Combined DDA and Application Cryptogram Generation.

	Card Holder verification
Cardholder verification checks that the person using the card is the cardholder. The card contains a list of verification methods that it supports, and the conditions under which they should be applied. The terminal must navigate through this list and attempt the first method it finds for which the condition is met. If a method fails, the terminal must check whether additional methods are allowed. For example, a list might contain: online PIN (if unattended cash), offline PIN (if supported), signature (always).


	Processing Restrictions
Processing restrictions allow the terminal to determine the compatibility of the applications on the card and terminal. This involves checking if their Application Version Numbers match, if the card application is expired or pre-valid, and whether the Application Usage Control (AUC) permits the current transaction to be performed.

	Terminal Risk Management
To safeguard against fraudulent use, the terminal will manage the level of risk by requiring certain transactions to be online authorized that would otherwise have been authorized locally. This involves comparing the transaction amount against floor limits, and detecting when the number of consecutive offline transactions on a card has reached a defined limit. In addition, offline-capable terminals will also randomly select certain transactions to go online.

Terminal risk management may be performed at any time after Read Application Data but before issuing the first GENERATE AC command.

Terminal risk management consists of: 
• Floor limit checking 
• Random transaction selection 
• Velocity checking 

Upon completion of terminal risk management, the terminal shall set the ‘Terminal risk management was performed’ bit in the TSI to 1.

	Terminal Action Analysis
The terminal will analyse the results of the previous verification, authentication and risk steps and this will result in the terminal informing the card that it proposes to either seek online authorization of the transaction, or to complete it offline by accepting or declining the transaction local

Once terminal risk management and application functions related to a normal offline transaction have been completed, the terminal makes the first decision as to whether the transaction should be approved offline, declined offline, or transmitted online. 
• If the outcome of this decision process is to proceed offline, the terminal issues a GENERATE AC command to ask the ICC to return a TC. 
• If the outcome of the decision is to go online, the terminal issues a GENERATE AC command to ask the ICC for an Authorisation Request Cryptogram (ARQC). 
• If the decision is to reject the transaction, the terminal issues a GENERATE AC to ask for an Application Authentication Cryptogram (AAC). 

An offline decision made here is not final. If the terminal asks for a TC from the ICC, the ICC, as a result of card risk management, may return an ARQC or AAC.
Conditions of Execution: 
The terminal action analysis function is always performed. 
Sequence of Execution: 
The terminal action analysis function is performed after terminal risk management and cardholder and/or merchant transaction data entry has been completed. It shall be performed prior to the first use of the GENERATE AC command.









































	Card Action Analysis
During the first Card Action Analysis step, the card will analyse the results of all the previous steps and this will result in the card requesting the terminal to either seek online authorization of the transaction, or to complete it offline by accepting or declining the transaction locally. This request may differ from the action that the terminal proposed following Terminal Action Analysis, but is subject to certain logic rules (e.g. the card is not permitted to request offline acceptance of the transaction if the terminal proposed online authorization).

A CDOL is a list of data that the card requires during Card Action Analysis, 
and there are 2 different CDOL that may be required during the course of a transaction. 
CDOL1 is used during the first card action analysis, and if a second card action analysis is required then CDOL2 is used.The terminal uses the DOL processing rules to format the requested data and then sends it to the card in the Generate Application Cryptogram request.

There are 3 type of cryptogram that can be generated by the card:
1.	*AAC-----Application Authentication cryptogram---txn declined
2.	*ARQC---Authorisation Request Cryptogram---------online auth. requested
3.	*TC-----Txn Certificate---------------------------Txn Approved.
Generate AC Command
CLA  '80'	INS  'AE' 	P1-   Ref Ctrl Parameter		P2-  '00'
Lc     Var		DATA:CDOL 			Le'00'
response APDU is a constructed data object with tag equal to '77'.
note The CCD-compliant application shall not request that the terminal send an advice message to the issuer.The CCD-compliant application shall not contain a TDOL. The CCD-compliant application shall not request the terminal to generate a TC Hash Value (that is, tag '98' shall not be included in CDOL1 or CDOL2).
The transaction-related data is depending on a Card Risk Management Data Object List 1(CDOL1).This CDOL is given by the card. It contains the tag name and length of the expected data.
Card Risk Management Data Object List 1 (CDOL1): 9F02069F030695055F2A029A039C019F37049F4C089F4502
  // Tag - Length - Meaning
  9f02 - 06 - Authorised amount of the transaction (excluding adjustments)
  9f03 - 06 - Secondary amount associated with the transaction representing a cashback amount
  95 - 05 - Terminal Verification Results
  5f2a - 02 - Transaction Currency Code
  9a - 03 - Transaction Date
  9c - 01 - Transaction Type
  9f37 - 04 - Unpredictable Number
  9f4c - 08 - ICC Dynamic Number
  9f45 - 02 - Data Authentication Code
00 B2 01 04 26



First we create an ByteString which corresponds to the CDOL1 above.
To obtain the ICC Dynamic Number we send a Get Challenge command to the card. The card will return an 8-byte unpredictable number.
var authorisedAmount = new ByteString("000000000001", HEX);
var secondaryAmount = new ByteString("000000000000", HEX);
var tvr = new ByteString("0000000000", HEX);
var transCurrencyCode = new ByteString("0978", HEX);
var transDate = new ByteString("090730", HEX);
var transType = new ByteString("21", HEX);
var unpredictableNumber = crypto.generateRandom(4);
var iccDynamicNumber = card.sendApdu(0x00, 0x84, 0x00, 0x00, 0x00);
var DataAuthCode = e.cardDE[0x9F45];

var Data = authorisedAmount.concat(secondaryAmount).concat(tvr).concat(transCurrencyCode).concat(transDate).concat(transType).concat(unpredictableNumber).concat(iccDynamicNumber).concat(DataAuthCode); 
Then we set P1 to '40', to request an Transaction Certificate (offline transaction) and execute the Generate AC command.
var p1 = 0x40;

var generateAC = card.sendApdu(0x80, 0xAE, p1, 0x00, Data, 0x00);
The Generate AC command was succesful if the card returns SW1/SW2=9000.
96 C: 00 84 00 00 - GET CHALLENGE      Le=0 
   R: SW1/SW2=9000 (Normal processing: No error) Lr=8
      0000  8D 51 F4 6C 9F 40 5F 71                          .Q.l.@_q
96 C: 80 AE 40 00 - UNKNOWN_INS Lc=37 
      0005  00 00 00 00 00 01 00 00 00 00 00 00 00 00 00 00  ................
      0015  00 09 78 09 07 30 21 2C 76 4F 65 8D 51 F4 6C 9F  ..x..0!,vOe.Q.l.
      0025  40 5F 71 D1 79                                   @_q.y
      Le=0 
   R: SW1/SW2=9000 (Normal processing: No error) Lr=32
      0000  77 1E 9F 27 01 80 9F 36 02 02 13 9F 26 08 2D F3  w..'...6....&.-.
      0010  83 3C 61 85 5B EA 9F 10 07 06 84 23 00 31 02 08  .










	Online offline decision
The terminal must perform the action that the card requested during card action analysis.
Format 1, Tag '80'
The data object is a primitive data object. The value field consists of a concatenation of data objects without tag and length bytes.
The data objects are:
• Cryptogram Information Data (CID)
• Application Transaction Counter (ATC)
• Application Cryptogram (AC)
• Issuer Application Data (IAD) (optional data object)
Format 2, Tag '77'
The data object is a constructed data object. The value field contains TLV coded data objects. Mandatory data objects are:
•  Tag: '9F27' - Cryptogram Information Data (CID)
•  Tag: '9F36' - Application Transaction Counter (ATC)
•  Tag: '9F26' - Application Cryptogram (AC)
The decision will be send to the terminal in the response message of the Generate AC command.
R: SW1/SW2=9000 (Normal processing: No error) Lr=32
      0000  77 1E 9F 27 01 80 9F 36 02 02 13 9F 26 08 2D F3  w..'...6....&.-.
      0010  83 3C 61 85 5B EA 9F 10 07 06 84 23 00 31 02 08  .<="" pre="">
This is a format 2 response message to the Generate AC command.
It contains an ARQC which means that the transaction should be proceed online.Completion
Offline Transaction
If the card answers on a Generate AC command with a TC, the transaction will be completed offline.
Online Transaction.During the online processing the issuer can review and authorise or reject transactions.The transaction will be completed with a second Generate AC command, sending either a TC or an AAC.



	Online Processing
Online processing enables the card issuer to analyse the transaction details and decide whether it wishes to authorise or reject the transaction. This allows the issuer to check the account status and apply criteria based upon acceptable limits of risk defined by the card issuer, the payment scheme and the acquirer. If no valid response is received from the host (e.g. due to communications failure) then the terminal is required to perform additional Terminal Action Analysis to manage the increased level of risk, and this will result in the terminal informing the card that it proposes to either accept or decline the transaction locally.
	Second Card analysis
During the second Card Action Analysis step, after online processing has been performed, the card will analyse the result of the online processing and will authenticate data received from the card issuer. This will result in the card requesting the terminal to complete the transaction by either accepting or declining the transaction. This request may differ from the result of online processing, but is subject to certain logic rules (e.g. the card is not permitted to request acceptance of the transaction if the host declined the payment).

	Transaction Completed.
When the card processing has been completed, the card may be removed. If the transaction has been authorized then payment can be submitted for settlement and any goods or services can be provided.


