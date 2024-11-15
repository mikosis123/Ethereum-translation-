###(Restricting Access) መዳረሻን መገደብ  
 መዳረሻን መገደብ ለውሎች የተለመደ ንድፍ ነው።የትኛውንም ሰው ወይም ኮምፒውተር የግብይቶችዎን ይዘት ወይም የውል ሁኔታዎን እንዳያነቡ ፈጽሞ ሊገድቡ እንደማይችሉ ልብ ይበሉ። ምስጠራን(encryption) በመጠቀም ትንሽ ከባድ ማድረግ ይችላሉ,ነገር ግን ውልዎ መረጃን እንዲያነብ ከታሰበ ሁሉም ሰው እንዲሁ ያነበዋል።

    ወደ ውልዎ ሁኔታ የማንበብ መዳረሻን በሌሎች ኮንትራቶች መገደብ ይችላሉ።ይህ በእውነቱ ነባሪው ነው, የውል ሁኔታ ተለዋዋጮችዎን ይፋዊ(public )ካላወጁ በስተቀር።

በተጨማሪም፣ በውልዎ ሁኔታ ላይ ወይም የውልዎን ተግባራት ማን ማሻሻያ ማድረግ እንደሚችል መገደብ ይችላሉ። ይህ ክፍል ስለ እሱ ነው።

የተግባር ማሻሻያዎች( function modifiers)ን መጠቀም እነዚህን ገደቦች በጣም ሊነበቡ የሚችሉ ያደርጋቸዋል።

// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

contract AccessRestriction {
// እነዚህ በግንባታ ደረጃ ላይ ይሰየማሉ
// ይህም `msg.sender` የውል ፈጣሪ መለያ ነው

    address public owner = msg.sender;
    uint public creationTime = block.timestamp;

    // አሁን ይህ ውል አንድ ላይ ሊያመነጭ የሚችለውን የስህተት  ዝርዝር በልዩ አስተያየቶች ውስጥ ከጽሑፋዊ ማብራሪያ ጋር ይከተላል።



    /// ላኪ ለዚህ ተግባር አልተፈቀደለትም።
    error Unauthorized();

    ///ተግባሩ በጣም ቀደም ብሎ የተጠራ።

    error TooEarly();

    /// ከተግባር ጥሪ ጋር የተላከ በቂ ኤተር(Ether) የለም።
    error NotEnoughEther();

    // መቀየሪያዎች የአንድን ተግባር አካል ለመለወጥ ጥቅም ላይ ሊውሉ ይችላሉ።
    // ይህ መቀየሪያ ጥቅም ላይ ከዋለ, ተግባሩ ከተወሰነ አድራሻ ከተጠራ ብቻ የሚያልፍ ማረጋገጫ ያዘጋጃል።
    modifier onlyBy(address account)
    {
        if (msg.sender != account)
            revert Unauthorized();
        // ይህንን ምልክት አይርሱ"_;"! ይህ ምልክት መቀየሪያው

ጥቅም ላይ ሲውል ዋነኛውን የተግባር አካል ይተካዋል

        _;
    }

    /// አዲሱን የዚህ ውል ባለቤት `newOwner` የሥራ ባለቤት አድርግ።
    function changeOwner(address newOwner)
        public
        onlyBy(owner)
    {
        owner = newOwner;
    }

    modifier onlyAfter(uint time) {
        if (block.timestamp < time)
            revert TooEarly();
        _;
    }

    /// የባለቤትነት መረጃን ሰርዝ.
    /// ውሉ ከተፈጠረ ከ 6 ሳምንታት በኋላ ብቻ ሊጠራ ይችላል።
    function disown()
        public
        onlyBy(owner)
        onlyAfter(creationTime + 6 weeks)
    {
        delete owner;
    }

    // ይህ መቀየሪያ ከተግባር ጥሪ ጋር የተያያዘ የተወሰነ ክፍያ ያስፈልገዋል።
    // ጥሪ ድራጊው ብዙ ከላከ/ች ለእሱ ወይም ለእሷ ተመላሽ ይደረጋል፣ ግን ከተግባር አካል በኋላ ብቻ ነው።
    // ይህ ከ Solidity ስሪት 0.4.0 በፊት አደገኛ ነበር
    // ከ `_;` በኋላ ያለውን ክፍል መዝለል ይቻል ነበር።

    modifier costs(uint amount) {
        if (msg.value < amount)
            revert NotEnoughEther();

        _;
        if (msg.value > amount)
            payable(msg.sender).transfer(msg.value - amount);
    }

    function forceOwnerChange(address newOwner)
        public
        payable
        costs(200 ether)
    {
        owner = newOwner;
        // የአንዳንድ ሁናቴ ምሳሌ
        if (uint160(owner) & 0 == 1)
     // ይህ ከስሪት 0.4.0 በፊት ለ Solidity ገንዘብ አልተመለሰም።
            return;
        // ከመጠን በላይ የተከፈሉ ክፍያዎችን ይመልሱ።
    }

}

የተግባር ጥሪዎችን ተደራሽነት የሚገድብበት የበለጠ ልዩ መንገድ በሚቀጥለው ምሳሌ ይብራራል።
