<script setup>
  import { ref, onMounted, onBeforeUnmount } from "vue";

  import axios from "axios";

  import { Notyf } from "notyf";
  import "notyf/notyf.min.css";

  const notyf = new Notyf();

  const WEB3FORM_ACCESS_KEY = "9ecbb1ed-6b38-46d1-9eb1-911997942210";

  const emailSubject = "New message from Portfolio Contact Form";

  const SITE_KEY = "6LcFkBouAAAAAETGVrVHG-CeQxHcySbclvsEnDwQ";

  const name = ref("");
  const email = ref("");
  const message = ref("");

  const isLoading = ref(false);

  const recaptchaContainer = ref(null);
  const recaptchaWidgetId = ref(null);
  const recaptchaToken = ref("");

  const onRecaptchaSuccess = (token) => {
    recaptchaToken.value = token;
  };

  const onRecaptchaExpired = () => {
    recaptchaToken.value = "";
  };

  const renderRecaptcha = () => {
    if (!window.grecaptcha) {
      console.error("reCAPTCHA not loaded");
      return;
    }

    recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
      sitekey: SITE_KEY,
      size: "normal",
      callback: onRecaptchaSuccess,
      "expired-callback": onRecaptchaExpired,
    });
  };

  const resetRecaptcha = () => {
    if (recaptchaWidgetId.value !== null) {
      window.grecaptcha.reset(recaptchaWidgetId.value);
      recaptchaToken.value = "";
    }
  };

  onMounted(() => {
    const interval = setInterval(() => {
      if (window.grecaptcha && window.grecaptcha.render) {
        renderRecaptcha();
        clearInterval(interval);
      }
    }, 100);

    onBeforeUnmount(() => {
      clearInterval(interval);
    });
  });

  const sendForms = async (e) => {
    e.preventDefault();

    if (!recaptchaToken.value) {
      notyf.error("Please verify that you are not a robot.");
      return;
    }

    isLoading.value = true;

    try {
      const response = await axios.post("https://api.web3forms.com/submit", {
        access_key: WEB3FORM_ACCESS_KEY,
        name: name.value,
        email: email.value,
        subject: emailSubject,
        message: message.value,
      },{
        headers: {
          'Content-Type': "application/json",
          Accept: "application/json",
        },
      });

      if (response.data.success) {
        notyf.success("Message sent successfully!");
        name.value = "";
        email.value = "";
        message.value = "";
        resetRecaptcha();
      } else {
        notyf.error("Failed to send message. Please try again.");
      }
    } catch (error) {
      notyf.error("Something went wrong. Please try again later.");
    } finally {
      isLoading.value = false;
    }
  };

</script>

<template>
            <!-- contact -->
            <h1 class="text-center my-4 pt-5" id="contact">Contact</h1>
            <div class="contact-section">
                <div class="row align-items-center mt-4">
                    <div class="col-md-6 map-container">
                        <iframe id="gmap_canvas" src="https://maps.google.com/maps?q=centro%20escolar%20university%20manila&t=&z=13&ie=UTF8&iwloc=&output=embed" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe>
                    </div>
                    <div class="col-md-6">
                        <form @submit.prevent="sendForms">
                            <div class="mb-3">
                                <input v-model="name" type="text" class="form-control contact-form-control" placeholder="First Name M.I. Last Name">
                            </div>
                            <div class="mb-3">
                                <input v-model="email" type="email" class="form-control contact-form-control" placeholder="Email">
                            </div>
                            <div class="mb-3">
                                <textarea v-model="message" class="form-control contact-form-control" rows="6" placeholder="Message"></textarea>
                            </div>
                            <div class="mb-3" ref="recaptchaContainer"></div>
                            <div class="form-footer">
                                <div class="social-icons">
                                    <a href="https://www.linkedin.com/in/charles-babbage-8291a6211/" id="linkedin"><i class="fab fa-linkedin"></i></a>
                                    <a href="https://gitlab.com/cbabbage0991" id="gitlab"><i class="fab fa-gitlab"></i></a>
                                    <a href="https://github.com/cbabbage0991" id="github"><i class="fab fa-github"></i></a>
                                </div>
                                <button type="submit" class="submit-btn pl-5 pr-5" :disabled="isLoading">{{ isLoading ? "Sending..." : "Submit" }}</button>
                            </div>
                        </form>

                    </div>
                </div>
            </div>
</template>
