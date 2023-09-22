<script>
// @ts-nocheck
  import { onMount } from 'svelte';
  import InputForm from "../../components/InputForm.svelte";
  import Button from "../../components/Button.svelte";
  import ErrorMessage from "../../components/ErrorMessage.svelte";
  import { goto } from "$app/navigation";
  import { getAuth, signInWithPhoneNumber, RecaptchaVerifier } from "firebase/auth";
  import { pageType } from '../../store';

  let phoneNumber = "";
  let alertShow = false;
  let alertContent = "";
  /**
     * @type {import("@firebase/auth").ApplicationVerifier}
     */
  let recaptchaVerifier;

  onMount(() => {
    setTimeout(() => {
      const auth = getAuth();
      recaptchaVerifier = new RecaptchaVerifier(auth, 'recaptcha-container', {
        size: 'invisible', // Optional configuration options
        callback: () => {
          // Handle reCAPTCHA response
        },
        'expired-callback': () => {
          // Handle expired reCAPTCHA
        }
      });
      let type;
      pageType.subscribe(value => {
        type = value;
      });
      console.log(type);
    }, 100)
  });

  const phoneVerify = () => {
    // goto("/auth/confirm");
    // return;
    const auth = getAuth();
    auth.languageCode = 'it';
    signInWithPhoneNumber(auth, phoneNumber, recaptchaVerifier)
      .then((confirmationResult) => {
        // @ts-ignore
        window.confirmationResult = confirmationResult;
        goto("/auth/confirm");
      }).catch((error) => {
        alertContent = `${error.name}: ${error.code}`;
        alertShow = true;
      });
  }

  // @ts-ignore
  const handleInputChange = (e) => {
    phoneNumber = e.target.value;
  }


</script>

<div class="grid grid-cols-1">
  <p class="text-4xl font-bold text-center text-main mb-24 ">Je suis un talent</p>
  <div class="mb-24">
    <ErrorMessage show={alertShow} content={alertContent}/>
    <InputForm label="Téléphone" value={phoneNumber} on:change={handleInputChange} on:focus={() => alertShow=false}/>
  </div>
  <div id="recaptcha-container"></div>
  <Button on:click={() => phoneVerify()}>Suivant</Button>

</div>