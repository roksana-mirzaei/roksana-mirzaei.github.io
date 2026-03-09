---
layout: post
title:  "VirtualTryOn"
date:   2026-01-23 21:15:40 +0000
categories: []
image: /assets/images/VirtualTryOn/latent_diffiusion_model.png
---

**Authors:** Roksana Mirzaei, Gbenga Ilesanmi, Jerson Rojas Ortega, Soujanya Joshi

**Github:** [https://github.com/roksana-mirzaei/VirtualTryOnStyleStudio](https://github.com/roksana-mirzaei/VirtualTryOnStyleStudio)

<div style="font-size: 1.07em; color: #2563eb; margin-bottom: 1.2em;">
<strong>Note:</strong> This is a personal development collaboration research project with Roksana Mirzaei, Gbenga Ilesanmi, Jerson Rojas Ortega, Soujanya Joshi. The project is conducted outside of daily work, solely for learning and research purposes.
</div>

In 2025, virtual try-on became the next major trend in retail media. Many companies—including Google, H&M, and Zara—developed or integrated their own virtual try-on solutions to improve customer shopping experiences, drive sales, and reduce returns. Ray-Ban lets you virtually try on glasses using your webcam's live feed, while L'Oréal helps you experiment with makeup and hair color through live webcam feeds or uploaded images. Hugo Boss created a 3D avatar so you can see how their garments would look on you. As you can see, there's been lots of exciting progress in this space!

<img src="/assets/images/VirtualTryOn/vton.png" alt="Virtual Try-On Example" style="width: 40%; max-width: 400px; display: block; margin: 20px auto;">

The goal of this virtual try-on is to recreate the feel of an in-store fitting room—giving shoppers a realistic preview of how specific garments look on them. By making sizing and style choices more visible and low-risk, it's expected to deepen engagement, support more confident purchases, and reduce returns and exchanges—ultimately lifting sales and revenue.

Our research shows that virtual try-ons encourage customers to explore a wider range of products and spend more time with brands—both in-store and online—driving higher sales. They also open future opportunities for smarter campaign targeting and richer behavioral analytics across channels.

For shoppers, the experience feels more intuitive and reassuring: it boosts satisfaction, reduces returns, and improves conversion. It meaningfully supports people with disabilities, strengthening relationships through more inclusive design. By giving users an immersive preview and more autonomy in decision-making, virtual try-ons simulate the feel of an in-store fitting room. The effect is especially strong with younger audiences, while also broadening the customer base to better include those with accessibility needs.

As a group of 4 working in the retail media industry and as machine learning fans, we were curious to implement our own. This is a personal development project outside of our daily work, implemented solely for learning purposes.

In this article, I'll explain how we implemented the ML driven virtual try‑on pipeline in our experience. To do this, we combined and assembled different open‑source libraries and components. The following libraries were used:

**Main Pre‑Requisite Libraries**

- [Detectron2](https://github.com/facebookresearch/detectron2): Facebook AI's detection framework. Backbone for DensePose, providing detection and instance segmentation used in the pose/body‑part pipeline.
- [DensePose](https://github.com/facebookresearch/DensePose): Dense human pose estimation on top of Detectron2. Maps person pixels to a 3D surface to segment body parts (torso, arms, legs, hands), so we know what to preserve vs. where to place garments.
- Diffusers (from Hugging Face): Stable Diffusion tooling for generation. Uses the `stable-diffusion-inpainting` model from ["booksforcharlie"](https://huggingface.co/booksforcharlie/stable-diffusion-inpainting), including VAE (AutoencoderKL), UNet, and a DDIM‑style scheduler for garment transfer and blending.
- Transformers (from Hugging Face): Model zoo utilities. Provides CLIP image processing/encoding and convenient pre‑trained model loading and feature extraction.
- Self‑Correction Human Parsing (SCHP): Semantic human segmentation (ATR/LIP schemes) to separate clothes, face, hair, accessories, etc., producing precise masks for the try‑on edits.
- PyTorch
- FastAPI

Before going ahead, I'll mention some of the challenges we came across while implementing this:

- **Computational Overhead:** Managing the heavy computational resources needed for the model.
- **Privacy Concerns:** Handling private user info (like photos) means strict GDPR compliance. Users need to trust their data is safe, whether it's processed on-device or sent to a server.
- **On-Device Model Challenges:** On-device models are safer but eat up device resources and need compression tricks (like quantization or pruning) to shrink size without losing accuracy.
- **Cloud Deployment Security:** Running models in the cloud means sending customer data off-device, which can introduce security risks.
- **Cost Management:** Balancing the costs of development and deployment.
- **Scalability:** Making sure the app scales and works across different devices, OSes, and browsers for maximum accessibility.
- **User Education:** Helping older customers who might find the tech overwhelming.
- **Compatibility Issues:** Not everyone has the up to date tech device, so we have to check smartphone accessibility.
- **Model Limitations:** Dealing with things like inaccurate garment coverage or weird extra parts. We're looking at defining key points and letting users manually fix errors.
- **Challenges wrapping clothes:** Getting clothes to realistically wrap and fit on different body types is still a tough nut to crack.

# **End-to-End Lifecycle**

This section outlines the key phases of the data journey, from image collection to final output.

**Image Requirements**

For best results, users must use a high-resolution, portrait photo showing their whole body facing forward, with arms at sides. Images must be JPEG, JPG, or PNG. If these criteria aren't met, the pipeline won't work.

**Data Lifecycle Phases**

1. **Image Preprocessing**
    - The image is resized to 768×1024 px, keeping proportions intact.
    - The background is removed (using *rembg*), leaving only the user on a plain white background.
    - The image is segmented into body parts (using *detectron2*) for accurate clothing placement.
    - *OpenPose* detects body keypoints (shoulders, elbows, hips, etc.), producing:
        - a JSON file with joint coordinates
        - a visual keypoint overlay
2. **AI Model Processing**
    - Temporary folders are created for inputs.
    - The try-on model takes 6 inputs:
        1. **User image (background removed):** This is users' photo with everything in the background erased. It's the main picture we use for the virtual try-on.
        2. **OpenPose image:** A version of the photo with dots and lines showing where joints are—like shoulders, elbows, and hips—so the model knows how the user is standing.
        3. **Keypoints JSON:** A file with the exact positions of the user’s body joints. It's like a map that helps the model figure out how to fit the clothes on the user.
        4. **Human parsing output:** An image where the user’s body is split into parts (like head, arms, legs), so the model knows where to put the clothes and what to leave alone.
        5. **Clothing image:** The picture of the clothing the user wants to try on, ready to be placed on original photo.
        6. **Masked clothing image:** The same clothing, but with just the garment showing—no background or extra bits—so it fits neatly onto the user’s image.
3. **Post-processing & Output**
    - The final image is saved as a JPG, and then all temporary files are deleted to protect user privacy.

No user images are stored beyond immediate processing—everything is securely deleted once the result is delivered.

<img src="/assets/images/VirtualTryOn/e2e_diagram.png" alt="Virtual Try-On Example" style="width: 80%; max-width: 800px; display: block; margin: 20px auto;">

# Latent Diffusion Models (LDMs)

Okay, now let's get to the fun part—the diffusion model! A diffusion model is a generative model that learns to transform noise into realistic images, kind of like watching a photo slowly come into focus.

**Key Components of LDMs**

- Latent Space: Compressed representation of images. Instead of working with high-dimensional images, LDMs operate in a smaller, more efficient space.
- Diffusion Process: A two-step process that adds noise to an image and then learns how to reverse it.
- Conditional Generation: LDMs generate images conditioned on specific inputs (e.g., a user's photo and clothing item).

**Advantages of LDMs for 2D Try-On**

- Efficiency: Operating in latent space reduces computational cost and increases speed.
- High Quality: The denoising process results in high-resolution, realistic images.
- Flexibility: Can handle different clothing types, user poses, and body shapes.

In our project, we use a pre-trained diffusion model called "stable-diffusion-inpainting" from the booksforcharlie page on Hugging Face. We don't fine-tune or retrain it—we use it as-is, straight from Hugging Face. In our pipeline, this model takes processed photo and the clothing image, then blends them together to create a realistic try-on result!

<img src="/assets/images/VirtualTryOn/latent_diffiusion_model.png" alt="Virtual Try-On Example" style="width: 80%; max-width: 800px; display: block; margin: 20px auto;">

First, both the user's photo and the clothing image are converted into a compressed "latent space" that makes it easier for the model to work with (step 1–5 in the diagram). The diffusion model then starts with random noise (step 6) and, step by step, transforms it into a clear image—using both the user's photo and the clothing as guides, along with a mask that helps blend the two together in just the right areas. During training, the model learns from pairs of user and clothing images, using reconstruction loss to make sure the generated image looks like the real one, and perceptual loss to ensure high visual quality by comparing features at different levels. Throughout the process, the model pays attention to the user's pose, body shape, and the style of the clothes, gradually "denoising" the image so the outfit looks natural. Once everything is blended, the model decodes this result back into a high-quality(step 9 to 11), realistic image of the user wearing the new clothes. As a final step, a safety checker makes sure the output is appropriate before it's shown to the user.

# Experimental Results

<img src="/assets/images/VirtualTryOn/samples.png" alt="Virtual Try-On Example" style="width: 80%; max-width: 800px; display: block; margin: 20px auto;">

# Model evaluation

To evaluate our pipeline we use below metrics:

**Structural Similarity Index Measure - SSIM:**

Measures the structure, luminance, and contrast of the predicted image against the input image. This score ranges from 0 (no similarity) to 1 (identical images).

**Peak Signal-To-Noise Ratio - PSNR:**

Expressed in decibels (*dB*), PSNR measures the ratio between the maximum possible pixel values of the input image and the predicted image — 30dB minimum.

**Mean Squared Error - MSE:**

Measures the average squared difference between corresponding pixels in the input image and the predicted image — the lower the better.

<img src="/assets/images/VirtualTryOn/Evaluations.png" alt="Virtual Try-On Example" style="width: 80%; max-width: 800px; display: block; margin: 20px auto;">



# References

[Guidance: a cheat code for diffusion models](https://sander.ai/2022/05/26/guidance.html)

[Breaking Down Stable Diffusion](https://medium.com/@shitijnigam/breaking-down-stable-diffusion-1cfe9d71ded3)

https://www.sciencedirect.com/science/article/abs/pii/S0262885624002014

**Most Bookmarked Paper on Virtual Try-On (2024):**

- [OOTDiffusion: Outfitting Fusion-Based Latent](https://paperswithcode.com/paper/ootdiffusion-outfitting-fusion-based-latent)

**Code for the First Instance of Dedicated Virtual Try-On Neural Network (2018):**

- [VITON GitHub Repository](https://github.com/xthan/VITON)

**Code with Samples Using VITON Dataset (2022):**

- [DeepFashion Try-On GitHub Repository](https://github.com/switchablenorms/DeepFashion_Try_On)

**Academic References:**

1. Radford, A., Kim, J.W., Hallacy, C., Ramesh, A., Goh, G., Agarwal, S., Sastry, G., Askell, A., Mishkin, P., Clark, J., Krueger, G. and Sutskever, I. (2021). Learning Transferable Visual Models From Natural Language Supervision. *arXiv:2103.00020 [cs]*. [online] Available at: [Learning Transferable Visual Models From Natural Language Supervision](https://arxiv.org/abs/2103.00020) .
2. ‌ Chen, M., Chen, X., Zhai, Z., Ju, C., Hong, X., Lan, J. and Xiao, S. (2024). *Wear-Any-Way: Manipulable Virtual Try-on via Sparse Correspondence Alignment*. [online] [arXiv.org e-Print archive](http://arxiv.org/) . doi:https://doi.org/10.48550/arXiv.2403.12965.
3. Yu, M., Ma, Y., Wu, L., Cheng, K., Li, X., Meng, L. and Chua, T.-S. (2024). *Smart Fitting Room: A One-stop Framework for Matching-aware Virtual Try-on*. [online] [arXiv.org e-Print archive](http://arxiv.org/) . doi:https://doi.org/10.1145/3652583.3658064.
4. Lee, S., Lee, S. and Lee, J. (n.d.). *Towards Detailed Characteristic-Preserving Virtual Try-On*. [online] Available at: https://openaccess.thecvf.com/content/CVPR2022W/CVFAD/papers/Lee_Towards_Detailed_Characteristic-Preserving_Virtual_Try-On_CVPRW_2022_paper.pdf [Accessed 8 Jul. 2024].
5. Blalock, J., Munechika, D., Karanth, H., Helbling, A., Mehta, P., Lee, S. and Chau, D.H. (2024). *Mobile Fitting Room: On-device Virtual Try-on via Diffusion Models*. [online] [arXiv.org e-Print archive](http://arxiv.org/) . doi:https://doi.org/10.48550/arXiv.2402.01877.
6. ‌Islam, T., Miron, A., Liu, X. and Li, Y. (2024). Deep Learning in Virtual Try-On: A Comprehensive Survey. [*IEEE Access*, pp.1–1. doi:https://doi.org/10.1109/access.2024.3368612.](https://ieeexplore.ieee.org/document/10443388)
7. *Official Hugo Boss Website*, Nov. 2023, [online] Available: [https://www.ray-ban.com](https://www.ray-ban.com/).
8. *Virtual Try On for Hair Colour & Makeup, L’Oréal Paris UK*, Nov. 2023, [online] Available: [Beauty just got smarter - introducing our newest online tools](https://www.loreal-paris.co.uk/virtual-try-on)
9. *Official Ray-Ban Website*, Nov. 2023, [online] Available: https://www.hugoboss.com/uk/all-brands/men/new-in/virtual-try-on/ .
10. The Grocer. (n.d.). M&S trialling augmented reality wayfinding app at Westfield Food Hall. [online] Available at: [M&S trialling augmented reality wayfinding app at Westfield Food Hall](https://www.thegrocer.co.uk/marks-and-spencer/mands-trialling-augmented-reality-wayfinding-app-at-westfield-food-hall/662434.article) [Accessed 8 Jul. 2024].
11. Engine Creative. (n.d.). *Tesco Discover Augmented Reality Publishing and Retail Strategy*. [online] Available at: https://www.enginecreative.co.uk/portfolio/augmented-reality-publishing-retail-strategy/ .
12. H&M (2022). HIGH FASHION MEETS VIRTUAL FANTASY IN H&M’S LATEST INNOVATION STORY. *about*. [online] 17 Nov. Available at: [HIGH FASHION MEETS VIRTUAL FANTASY IN H&M’S LATEST INNOVATION STORY](https://about.hm.com/news/general-news-2022/high-fashion-meets-virtual-fantasy-in-h-m-s-latest-innovation-st.html) .
