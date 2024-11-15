# Common Patterns( የተለመዱ ንድፎች )  

## ከውሎች ወጪ ማድረግ  
ከውጤት በኋላ ገንዘቦችን ለመላክ የሚመከረው ዘዴ የወጪ-ንድፍን (withdrawal pattern) መጠቀም ነው።  
ምንም እንኳን ኢተር (Ether)ን ለመላክ በጣም ሊታወቅ የሚችለው ዘዴ, በውጤቱ ምክንያት, ቀጥተኛ የዝውውር ጥሪ (transfer call) ቢሆንም, ይህ የደህንነት ስጋትን ስለሚያስከትል አይመከርም።  
ስለዚህ ጉዳይ በደህንነት ጉዳዮች ገጽ ላይ የበለጠ ማንበብ ይችላሉ።  

### ምሳሌ:  
የሚከተለው ምሳሌ, ውል ውስጥ በተግባር የሚፈፀም የወጪ ንድፍ ሲሆን, ግቡ ከፍተኛውን የተወሰነ ማካካሻ መላክ ነው።  
ለምሳሌ፣ ኤተር (Ether)፣ “በጣም ሀብታም (richest)” ለመሆን ወደ ውሉ የሚደረግ እንዲሆን።  

በሚከተለው ውል ውስጥ, ከአሁን በፊት በጣም ሀብታም ካልሆኑ, አሁን በጣም ሀብታም ከሆነው ገንዘብ ይቀበላሉ።  

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract WithdrawalContract {
address public richest;
uint public mostSent;

    mapping(address => uint) pendingWithdrawals;

/// የተላከው የኤተር(Ether) መጠን አሁን ካለው ከፍተኛ መጠን ከፍ ያለ  
 error NotEnoughEther();

    constructor() payable {
        richest = msg.sender;
        mostSent = msg.value;
    }

    function becomeRichest() public payable {
        if (msg.value <= mostSent) revert NotEnoughEther();
        pendingWithdrawals[richest] += msg.value;
        richest = msg.sender;
        mostSent = msg.value;
    }

    function withdraw() public {
        uint amount = pendingWithdrawals[msg.sender];

// እንደገና የመግባት ጥቃቶችን (reentrancy attacks) መከላከል ከመላክዎ በፊት በመጠባበቅ ላይ ያለውን ተመላሽ ገንዘብ ዜሮ ማድረግዎን ያስታውሱ።

        pendingWithdrawals[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }

}
```
ይህ ይበልጥ ሊታወቅ ከሚችለው የመላኪያ ንድፍ በተቃራኒ ነው፡-
```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract SendContract {
address payable public richest;
uint public mostSent;

/// የተላከው የኤተር(Ether) መጠን አሁን ካለው ከፍተኛ መጠን ከፍ ያለ አልነበረም
error NotEnoughEther();

    constructor() payable {
        richest = payable(msg.sender);
        mostSent = msg.value;
    }

    function becomeRichest() public payable {
        if (msg.value <= mostSent) revert NotEnoughEther();
        //ይህ መስመር ችግር ሊፈጥር ይችላል (ከዚህ በታች ተብራርቷል)።.
        richest.transfer(msg.value);
        richest = payable(msg.sender);
        mostSent = msg.value;
    }

}
```
በዚህ ምሳሌ አንድ አጥቂ ውሉን ወደማይጠቅም ሁኔታ ሊያጠምደው የሚችለው ሃብታሞች (richest ) የውል አድራሻ እንዲሆኑ በማድረግ የመቀበል ወይም የመመለሻ ተግባር ያልተሳካለት መሆኑን (ለምሳሌ revert () በመጠቀም ወይም ወደ እነርሱ ከተላለፈው 2300 የጋዝ ክፍያ በላይ ብቻ በመመገብ) ልብ ይበሉ።በዚህ መንገድ፣ ወደ “የተመረዘ” ውል ገንዘብ ለማድረስ ዝውውር በተጠራ ቁጥር አይሳካም እና ስለዚህ "በጣም ሪችስት" አይሳካም ፣ ውሉ ለዘላለም ታስሯል።

በአንጻሩ፣ ከመጀመሪያው ምሳሌ የወጪ-ንድፍን ከተጠቀሙ፣ አጥቂው የራሱን ወይም የራሷን ወጪ እንዲከሽፍ ብቻ ነው እንጂ የተቀሩትን የውል ስራዎችን አይደለም።
