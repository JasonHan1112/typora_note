# Kernel Mode Setting

## Introduction

- Drivers must initialize the mode setting core by calling drmm_mode_config_init() on the DRM device. The function initializes the struct drm_device mode_config field and never fails. Once done, mode configuration must be setup by initializing the following fields.

- The basic object structure KMS presents to userspace is fairly simple. Framebuffers (represented by struct drm_framebuffer, see Frame Buffer Abstraction) feed into planes. Planes are represented by struct drm_plane, see Plane Abstraction for more details. One or more (or even no) planes feed their pixel data into a CRTC (represented by struct drm_crtc, see CRTC Abstraction) for blending.

- For the output routing the first step is encoders (represented by struct drm_encoder, see Encoder Abstraction). Those are really just internal artifacts of the helper libraries used to implement KMS drivers. 

- The final, and real, endpoint in the display chain is the connector (represented by struct drm_connector, see Connector Abstraction). Connectors can have different possible encoders, but the kernel driver selects which encoder to use for each connector. The use case is DVI, which could switch between an analog and a digital encoder. Encoders can also drive multiple different connectors. There is exactly one active connector for every active encoder.

## CRTC Abstraction

- It receives pixel data from drm_plane and blends them together. The drm_display_mode is also attached to the CRTC, specifying display timings. On the output side the data is fed to one or more drm_encoder, which are then each connected to one drm_connector.
