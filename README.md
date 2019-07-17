# Pay-Card
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Mini App</title>
    <style>
      body {
        margin: 0;
        padding: 1em;
        background-color:white;
      }
      
      [data-cart-info],
      [data-credit-card] {
        transform:scale(0.78);
    	margin-left: -3.4em;
      }
      
      [data-cc-info] input:focus,
      [data-cc-digits] input:focus {
        outline:none;
      }
      
      .mdc-card__primary-action,
      .mdc-card__primary-action:hover {
        cursor:auto;
        padding:20px;
        min-height:inherit;
      }
      
      [data-credit-card][data-card-type] {
        transition: width 1.5s;
        margin-left:calc(100% - 130px);
      }
      
      [data-credit-card].is-visa {
        background:linear-gradient(135deg, #622774 0%, #c53364 100%);
      }
      
      [data-credit-card].is-mastercard {
        background:linear-gradient(135deg, #65799b 0%, #5e2563 100%);
      }
      
      .is-visa [data-card-type],
      .is-mastercard [data-card-type] {
        width:auto;
      }
     
      input.is-valid,
      .is-invalid input {
        text-decoration:line-through;
      }
      
      ::placeholder{
        color:#fff;
      }
      
      [data-cart-info] span {
         display:inline-block;
         vertical-align:middle;
      }
     
      span.material-icons {
        font-size:150px;
      }
      [data-credit-card]{
        width:435px;
        min-height:240px;
        border-radius:10px;
        background-color:#5d6874;
      }
      [data-card-type]{
        display:block;
        width:120px;
        height:60px;
      }
      [data-cc-digits]{
        margin-top:2em;
      }
      [data-cc-digits] input{
        color:white;
        font-size:2em;
        line-height:2em;
        margin-right:0.5em;
        border: 0px;
        background:transparent;
      }
      
      [data-cc-info] {
        margin-top:1em;
      }
      [data-cc-info] input{
        color:white;
        font-size:1.2em;
        background:none;
        border: 0px;
      }
      [data-cc-info] input:last-of-type{
        padding-right:10px;
        float:right;
      }
      [data-pay-btn]{
        position:fixed;
        width:90%;
        border:1px solid;
        bottom:20px;
      }
    
    
    </style>
   </head>
  <body>
   <div data-cart-info>
    		<h1 class="mdc-typography--headline4">
       				<span class="material-icons">shopping_cart</span>
       				<span data-bill></span>
    		</h1>
   </div> 
  	<div data-credit-card class="mdc-card mdc-card--outlined">
    	<div class="mdc-card__primary-action">
       		<img data-card-type src="http://placehold.it/120x60.png?text=card"/>
       	<div data-cc-digits>
        	<input type="text" size="4" placeholder="----"/>
        	<input type="text" size="4" placeholder="----"/>
        	<input type="text" size="4" placeholder="----"/>
        	<input type="text" size="4" placeholder="----"/>
      </div>
      <div data-cc-info>
        	<input type="text" size="20" placeholder="Name Surname"/>
        	<input type="text" size="6"placeholder="MM/YY"/>
	  </div>
   </div>
</div>
    <button class="mdc-button" data-pay-btn>Pay &amp; Checkout Now</button> 
  
    <script>
      const appState = {};
      
      const formatAsMoney = (amount, buyerCountry) => {
        
        const [countryBill] = countries.filter(country => country.country === buyerCountry);
        
        if(!countryBill){
          return amount.toLocaleString('en-US', {style: 'currency', currency:'USD'});
        }
        
        return amount.toLocaleString(`en-${countryBill.code}`, {style: 'currency', currency: countryBill.currency});
      }
      
      const expiryDateFormatIsValid = (target) => {
        
        if (Date.parse(target.value) !== NaN ){
          return true;
        }
        
        return false;
      }
      
      const flagIfInvalid = (field, isValid) => {
        
        if(isValid){
          return field.classList.remove('is-invalid');
        }
        
        return field.classList.add('is-invalid');
      }
      
      const detectCardType = ({ target }) => {
        const cardCondition = /4[0-9]/
        
        if(cardCondition.test(target.value)){
          document.querySelector('img[data-card-type]').src = supportedCards.visa;
          document.querySelector('[data-credit-card]').classList.add("is-visa");
          return "is-visa";
        }
        
        document.querySelector('img[data-card-type]').src = supportedCards.mastercard;
        document.querySelector('[data-credit-card]').classList.add("is-mastercard");
        return "is-mastercard";
        
      }
      
      const validateCardExpiryDate = ({ target }) => {
        
        
        const test = expiryDateFormatIsValid(target.value);
        
        flagIfInvalid(target, test)
        
        return test;
      }
      
      const expiryDateFormatIsValid = (target) => {
        
        const checkDate = Date.parse(target);
        const [mm,yy] = target.split("/");
        
        if (!isNaN(checkDate)){
          const test = new Date() < new Date('20'+yy+'-'+mm);
          return test;
        }
        
        return false;
      }
      
      const validateCardHolderName = ({ target }) => {
        const testCondition = /[a-z]{3}\s[a-z]{3}/i;
        let checkCondition = true;
        if(target.value.split(" ").length > 2){
          checkCondition = false;
        } else {
        checkCondition = testCondition.test(target.value);
        }
        const result = flagIfInvalid(target, checkCondition);
        
        return checkCondition;
      }
      
      const validateWithLuhn = (digits) => {
        if(digits.length === 16){
          
        let i = digits.length-1;
        let elementPosition = 1;
        let sum =0;
        
        while(i > -1) {
          const eachElement = parseInt(digits[i]);
          if( elementPosition % 2 === 0) {
            if ( eachElement * 2 <= 9){
              sum = sum + eachElement * 2;
            }else{
              sum = sum + (eachElement * 2 -9);
            }
          }else {
            sum = sum + eachElement;
          }
          i--;
          elementPosition++;
        }
          
          if(sum === NaN) {
            return false;
          }
        
          
        if (sum % 10 === 0){
          return true;
        }
          
        }
        return false;
      }
      
      const validateCardNumber = () => {
        
        const selectInput = document.querySelectorAll('[data-cc-digits] input');
        
        let combine = "";
        selectInput.forEach(eachInput => {
          return combine = combine + eachInput.value;
        });
        
         const result = validateWithLuhn(combine);
        
        if(result){
          document.querySelector('[data-cc-digits]').classList.remove('is-invalid');
        } else {
          document.querySelector('[data-cc-digits]').classList.add('is-invalid');
        }
        
        return result;
      }
      
      const uiCanInteract = () => {
        const selectInputFirstChild = document.querySelector("[data-cc-digits] input:first-child");
        selectInputFirstChild.addEventListener('blur', detectCardType);
        
        document.querySelector("[data-cc-info] input:first-child").addEventListener('blur', validateCardHolderName);
        
        document.querySelector("[data-cc-info] input:nth-child(2)").addEventListener('blur', validateCardExpiryDate)
        
        document.querySelector("button[data-pay-btn]").addEventListener('click', validateCardNumber);
        
        document.querySelector("[data-cc-digits]").focus();
      }
      
      const displayCartTotal = ({ results }) => {
        const [data] = results;
        const { itemsInCart, buyerCountry}= data;
        
        appState.items = itemsInCart;
        appState.country = buyerCountry;
        
        appState.bill = itemsInCart.reduce((total, {price, qty}) => total + (price * qty), 0)
        
        appState.billFormatted = formatAsMoney(appState.bill, buyerCountry);
        
        document.querySelector("span[data-bill]").textContent = appState.billFormatted;
        
        uiCanInteract();
      }
      
      const fetchBill = () => {
        const api = 'https://randomapi.com/api/006b08a801d82d0c9824dcfdfdfa3b3c'
        fetch(api)
        .then(response => {
          if(response.ok){
            return response.json();
          }
        })
        .then(data => {
          displayCartTotal(data);
        })
      }
      
      const supportedCards = {
        visa, mastercard
      };
      
      const countries = [
        {
          code: "US",
          currency: "USD",
          country: 'United States'
        },
        {
          code: "NG",
          currency: "NGN",
          country: 'Nigeria'
        },
        {
          code: 'KE',
          currency: 'KES',
          country: 'Kenya'
        },
        {
          code: 'UG',
          currency: 'UGX',
          country: 'Uganda'
        },
        {
          code: 'RW',
          currency: 'RWF',
          country: 'Rwanda'
        },
        {
          code: 'TZ',
          currency: 'TZS',
          country: 'Tanzania'
        },
        {
          code: 'ZA',
          currency: 'ZAR',
          country: 'South Africa'
        },
        {
          code: 'CM',
          currency: 'XAF',
          country: 'Cameroon'
        },
        {
          code: 'GH',
          currency: 'GHS',
          country: 'Ghana'
        }
      ];
      
      const startApp = () => {
        fetchBill();
      };

      startApp();
    </script>
  </body>
</html>
