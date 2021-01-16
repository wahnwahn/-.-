
   # "ว่านว่าน.ร้านหมากับแมว"
 
   ใน Assignmentที่ 3 นี้เป็นการนำ Project Pet-shop มาปรับปรุงให้เป็นแพลตฟอร์มสำหรับการรับอุปการะสุนัขและแมว ซึ่งโครงการนี้เป็นการร่วมมือกันระหว่างมูลนิธิสันติสุขเพื่อสุนัขและแมวจรจัด ตั้งอยู่ที่จังหวัดเชียงใหม่ เป็นองค์กรการกุศลที่รับช่วยเหลือสุนัขและแมวที่ได้รับการทอดทิ้งและได้รับบาดเจ็บมาทำการรักษา ช่วยเหลือและเลี้ยงดู ส่วนผู้ที่ใจบุญสามารถนำอาหารสุนัขและแมว แชมพูอาบน้ำมาบริจาคได้ตามที่ตั้งของมูลนิธิสันติสุขแห่งนี้  
 
   ว่านว่าน.ร้านหมากับแมว มีวัตถุประสงค์ในการก่อตั้งคือเพื่อหาบ้านให้กับสุนัขและแมว หลังจากที่ทางมูลนิธิให้ความช่วยเหลือรับอุปการะเลี้ยงดู ทำการรักษาสุนัขและแมวแล้ว ทางเราจึงพยายามที่จะหาบ้านให้เนื่องจากสุนัขและแมวยังต้องการความรักความอบอุ่นจากมนุษย์ ซึ่งทางว่านว่าน.ร้านหมากับแมวจึงเปิดดำเนินการขึ้น
 
  **Wahnwahn Kempinski** เป็นหนึ่งในผู้คิดค้นระบบการชำระด้วยเหรียญ Ethereum และเป็น 1 ในผู้ก่อตั้งว่านว่าน.ร้านหมากับแมว อีกทั้งยังเป็นผู้ใจบุญที่ให้ทุนช่วยเหลือสุนัขและแมวของมูลนิธิสันติสุขแห่งนี้อย่างบ่อยครั้ง โดยการใช้บล็อกเชนเพื่อให้เกิดความโปร่งใส โดยข้อมูลเหล่านี้จะถูกเก็บไว้ในบล็อกเชนตลอดไป   

## การวิเคราะห์และออกแบบ (Analysis & Design) 
ใช้ Smart contract ที่เขียนโดยภาษา Solidity ซึ่งเรา deploy บนบล็อกเชนส่วนบุคคล Ganache/Metamask ไปเชื่อมต่อกับ Front-end และ back-end  เพื่อให้แสดงผลออกมาเป็นเว็บไซต์ 

## การจัดทำ (Implementation)
## 1. สร้าง Smart Contract
     1.1. Adoption Smart Contract
       โดยไปที่ Visual Studio Code แล้วสร้างไฟล์ชื่อ Adoption.sol ในไดเร็กทอรี contracts โดยมีโค้ดดังนี้

pragma solidity ^0.5.0;

contract Adoption {
    address[16] public adopters;

    function adopt(uint petId) public returns (uint) {
        require(petId >= 0 && petId <=15);
        adopters[petId] = msg.sender;
        return petId;
    }

    function getAdopters() public view returns (address[16] memory) {
        return adopters;
    }
}
  1.2. compile และ migrate Smart Contracts โดยใช้คำสั่ง truffle compile
       ทำการสร้าง NEW WORKSPACE ใน Ganache โดยใช้ชื่อว่า NF507
  
  1.3. ทดสอบ Smart Contract
       โดยไปที่ Visual Studio Code สร้างไฟล์ TestAdoption.sol เพื่อทดสอบ Adoption.sol และบันทึกลงในไดเร็กทอรี test โดยมีโค้ดดังนี้

pragma solidity ^0.5.0;

import "truffle/Assert.sol";
import "truffle/DeployedAddresses.sol";
import "../contracts/Adoption.sol";

contract TestAdoption {
  // The address of the adoption contract to be tested
  Adoption adoption = Adoption(DeployedAddresses.Adoption());

  // The id of the pet that will be used for testing
  uint expectedPetId = 8;

  //The expected owner of adopted pet is this contract
  address expectedAdopter = address(this);

  function testUserCanAdoptPet() public {
    uint returnedId = adoption.adopt(expectedPetId);
    Assert.equal(returnedId, expectedPetId, "Adoption of the expected pet should match what is returned.");
  }

  // Testing retrieval of a single pet's owner
  function testGetAdopterAddressByPetId() public {
    address adopter = adoption.adopters(expectedPetId);
    Assert.equal(adopter, expectedAdopter, "Owner of the expected pet should be this contract");
  }

  // Testing retrieval of all pet owners
  function testGetAdopterAddressByPetIdInArray() public {
    // Store adopters in memory rather than contract's storage
    address[16] memory adopters = adoption.getAdopters();
    Assert.equal(adopters[expectedPetId], expectedAdopter, "Owner of the expected pet should be this contract");
  }
}

2. ออกแบบ UI เพื่อใช้เชื่อมต่อกับผู้ใช้
ส่วนติดต่อกับผู้ใช้ 


โดยใช้ไฟล์ src/index.html ในขณะที่ข้อมูลที่ใช้ในการแสดงผลจะถูกกำหนดโดยส่วน Backend


## 3. สร้าง Backend ที่สามารถเชื่อมต่อกับ Smart Contract
      แก้ไขไฟล์ src/js/app.js ให้มีโค้ดดังนี้

App = {
  web3Provider: null,
  contracts: {},

  init: async function() {
    // Load pets.
    $.getJSON('../pets.json', function(data) {
      var petsRow = $('#petsRow');
      var petTemplate = $('#petTemplate');

      for (i = 0; i < data.length; i ++) {
        petTemplate.find('.panel-title').text(data[i].name);
        petTemplate.find('img').attr('src', data[i].picture);
        petTemplate.find('.pet-breed').text(data[i].breed);
        petTemplate.find('.pet-age').text(data[i].age);
        petTemplate.find('.pet-location').text(data[i].location);
        petTemplate.find('.btn-adopt').attr('data-id', data[i].id);

        petsRow.append(petTemplate.html());
      }
    });

    return await App.initWeb3();
  },

  initWeb3: async function() {
    // Modern dapp browsers...
    if (window.ethereum) {
      App.web3Provider = window.ethereum;
      try {
        // Request account access
        await window.ethereum.enable();
      } catch (error) {
        // User denied account access...
        console.error("User denied account access")
      }
    }
    // Legacy dapp browsers...
    else if (window.web3) {
      App.web3Provider = window.web3.currentProvider;
    }
    // If no injected web3 instance is detected, fall back to Ganache
    else {
      App.web3Provider = new Web3.providers.HttpProvider('http://localhost:7545');
    }
    web3 = new Web3(App.web3Provider);

    return App.initContract();
  },

  initContract: function() {
    $.getJSON('Adoption.json', function (data) {
      // Get the necessary contract artifact file and instantiate it with @truffle/contract
      var AdoptionArtifact = data;
      App.contracts.Adoption = TruffleContract(AdoptionArtifact);

      // Set the provider for our contract
      App.contracts.Adoption.setProvider(App.web3Provider);

      // Use our contract to retrieve and mark the adopted pets
      return App.markAdopted();
    });
    return App.bindEvents();
  },

  bindEvents: function() {
    $(document).on('click', '.btn-adopt', App.handleAdopt);
  },

  markAdopted: function() {
    var adoptionInstance;

    App.contracts.Adoption.deployed().then(function (instance) {
      adoptionInstance = instance;

      return adoptionInstance.getAdopters.call();
    }).then(function (adopters) {
      for (i = 0; i < adopters.length; i++) {
        if (adopters[i] !== '0x0000000000000000000000000000000000000000') {
          $('.panel-pet').eq(i).find('button').text('Success').attr('disabled', true);
        }
      }
    }).catch(function (err) {
      console.log(err.message);
    });
  },

  handleAdopt: function(event) {
    event.preventDefault();

    var petId = parseInt($(event.target).data('id'));

    var adoptionInstance;

    web3.eth.getAccounts(function (error, accounts) {
      if (error) {
        console.log(error);
      }

      var account = accounts[0];

      App.contracts.Adoption.deployed().then(function (instance) {
        adoptionInstance = instance;

        // Execute adopt as a transaction by sending account
        return adoptionInstance.adopt(petId, { from: account });
      }).then(function (result) {
        return App.markAdopted();
      }).catch(function (err) {
        console.log(err.message);
      });
    });
  }

};

$(function() {
  $(window).load(function() {
    App.init();
  });
});

## 4. ติดตั้ง MetaMask

## 5. Run Program
run Backend โดยใช้คำสั่ง

    npm run dev
 
จากนั้นเปิด Firefox ที่ URL ดังนี้ http://localhost:3000
![Front](https://user-images.githubusercontent.com/75904103/104817482-1bb0c980-5854-11eb-9d60-1511f2a77699.PNG)
    
ระบบจะเชื่อมต่อไปยัง Wallet
![CF](https://user-images.githubusercontent.com/75904103/104817477-1784ac00-5854-11eb-8b8b-18cecaf56cda.PNG)
    
โปรดสังเกตุ ก่อนที่จะมีการอุปการะ ปุ่มด้านล่างจะขึ้นว่า "Adopt" เป็นตัวหนังสือสีเข้มอยู่ เมื่อมีคนกด Adopt ตัวหนังสือจะจางลง
![Success](https://user-images.githubusercontent.com/75904103/104817486-20757d80-5854-11eb-963f-7433743e1fbf.PNG)

ภาพแสดงธุรกรรมที่เกิดขึ้น
![TR](https://user-images.githubusercontent.com/75904103/104817476-15bae880-5854-11eb-8351-c80688e799b5.PNG)
