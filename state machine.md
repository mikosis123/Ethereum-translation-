# (State machine) የሁኔታ ማሽን

ውሎች ብዙውን ጊዜ እንደ ሁኔታ ማሽን ይሠራሉ, ይህም ማለት የተለየ ባህሪ ያላቸው ወይም የተለያዩ ተግባራት ሊጠሩ የሚችሉባቸው የተወሰኑ ደረጃዎች አሏቸው።የተግባር ጥሪ ብዙውን ጊዜ ውሉ ያለበትን ደረጃ ያበቃል እና ውሉን ወደ ቀጣዩ ደረጃ ይሸጋገራል (በተለይ የውሉ ሞዴሎች መስተጋብር(interaction) ከሆነ)። በተጨማሪም አንዳንድ ደረጃዎች በተወሰነ ጊዜ ውስጥ በራስ-ሰር(automatically) መድረሳቸው የተለመደ ነው።

ለዚህ ምሳሌ የሚሆነው በደረጃ “ዓይነ ስውር ጨረታዎችን በመቀበል” የሚጀምር ዓይነ ስውር የጨረታ ውል ነው፣ ከዚያም ወደ “የሚሸጋገር ጨረታዎች” በ “የተጠናቀቀ የጨረታ ውጤት ” ይወሰናል።

የተግባር ማሻሻያዎችን በዚህ ሁኔታ ውስጥ ሁኔታዎችን ለመቅረጽ እና የውሉን የተሳሳተ አጠቃቀም ለመከላከል ጥቅም ላይ ሊውል ይችላል።

### ምሳሌ

በሚከተለው ምሳሌ, የ atStage መቀየሪያ ተግባሩ በተወሰነ ደረጃ ላይ ብቻ ሊጠራ እንደሚችል ያረጋግጣል።

አውቶማቲክ የጊዜ ሽግግሮች የሚስተናገዱት በመቀየሪያ የጊዜ ሽግግር(timedTransitions) ሲሆን ይህም ለሁሉም ተግባራት ጥቅም ላይ መዋል አለበት።

ማስታወሻ:-የመቀየሪያ ቅደም-ትከተል: atStage ከጊዜ ሽግግር(timedTransition) ጋር ከተጣመረ፣ ከኋለኛው በኋላ መጥቀስዎን ያረጋግጡ ስለዚህ አዲሱ ደረጃ ግምት ውስጥ ይገባል.

በመጨረሻም የመቀየሪያው ሽግግር ቀጣይ ተግባሩ ሲያልቅ በራስ-ሰር(automatically) ወደ ቀጣዩ ደረጃ ለመሄድ ሊያገለግል ይችላል።

ማስታወሻ:-መቀየሪያ ሊገለበጥ ይችላል: ይህ ከስሪት 0.4.0 በፊት በብቸኝነት ላይ ብቻ ነው የሚሰራው፡መቀየሪያዎች የሚተገበሩት በቀላሉ ኮድን በመተካት እንጂ የተግባር ጥሪን በመጠቀም ስላልሆነ፣ በሽግግሩ ውስጥ ያለው ኮድ ቀጣይ መቀየሪያ ተግባሩ ራሱ መመለሻን ከተጠቀመ ሊዘለል ይችላል።ያንን ማድረግ ከፈለጉ፣ ከእነዚያ ተግባራት ወደ ቀጣዩ ደረጃ በእጅ(manually) መደወልዎን ያረጋግጡ።ከስሪት 0.4.0 ጀምሮ፣ ተግባሩ በግልፅ ቢመለስም የመቀየሪያ ኮድ ይሰራል።
```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract StateMachine {
enum Stages {
AcceptingBlindedBids,
RevealBids,
AnotherStage,
AreWeDoneYet,
Finished
}
/// በዚህ ጊዜ ተግባር ሊጠራ አይችልም.
error FunctionInvalidAtThisStage();

    // ይህ የአሁን ያለንበት ደረጃ ነው.
    Stages public stage = Stages.AcceptingBlindedBids;

    uint public creationTime = block.timestamp;

    modifier atStage(Stages stage_) {
        if (stage != stage_)
            revert FunctionInvalidAtThisStage();
        _;
    }

    function nextStage() internal {
        stage = Stages(uint(stage) + 1);
    }

    // በጊዜ የተያዙ ሽግግሮችን ያከናውኑ። በመጀመሪያ ይህንን ማሻሻያ መጥቀስዎን ያረጋግጡ,አለበለዚያ ጠባቂዎቹ አዲሱን ደረጃ ግምት ውስጥ አያስገቡም.
    modifier timedTransitions() {
        if (stage == Stages.AcceptingBlindedBids &&
                    block.timestamp >= creationTime + 10 days)
            nextStage();
        if (stage == Stages.RevealBids &&
                block.timestamp >= creationTime + 12 days)
            nextStage();
        // ሌሎቹ ደረጃዎች ግብይት በግብይት ላይ ይሸጋገራሉ
        _;
    }

    // የማሻሻያዎቹ ቅደም ተከተል እዚህ አስፈላጊ ነው!
    function bid()
        public
        payable
        timedTransitions
        atStage(Stages.AcceptingBlindedBids)
    {
        // ያንን እዚህ አንተገብርም።
    }

    function reveal()
        public
        timedTransitions
        atStage(Stages.RevealBids)
    {
    }

    // ይህ መቀየሪያ ተግባሩን ከጨረሰ በኋላ ወደሚቀጥለው ደረጃ ይሄዳል።
    modifier transitionNext()
    {
        _;
        nextStage();
    }

    function g()
        public
        timedTransitions
        atStage(Stages.AnotherStage)
        transitionNext
    {
    }

    function h()
        public
        timedTransitions
        atStage(Stages.AreWeDoneYet)
        transitionNext
    {
    }

    function i()
        public
        timedTransitions
        atStage(Stages.Finished)
    {
    }

}
```
