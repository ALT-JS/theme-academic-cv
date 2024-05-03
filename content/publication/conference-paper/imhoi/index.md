---
title: 'I’M HOI: Inertia-aware Monocular Capture of 3D Human-Object Interactions'

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here
# and it will be replaced with their full name and linked to their profile.
authors:
  - Chengfeng Zhao
  - Juze Zhang
  - admin
  - Ziwei Shan
  - Junye Wang
  - Jingyi Yu
  - Jingya Wang
  - Lan Xu

# Author notes (optional)
author_notes:
  - ''
  - ''
  - ''
  - ''
  - ''
  - ''
  - ''
  - 'Corresponding author'

date: '2023-11-18T00:00:00Z'
doi: ''

# Schedule page publish date (NOT publication's date).
publishDate: '2017-01-01T00:00:00Z'

# Publication type.
# Accepts a single type but formatted as a YAML list (for Hugo requirements).
# Enter a publication type from the CSL standard.
publication_types: ['paper-conference']

# Publication name and optional abbreviated publication name.
publication: In *IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2024.*
publication_short: In *CVPR2024*

abstract: We are living in a world surrounded by diverse and “smart” devices with rich modalities of sensing ability. Conveniently capturing the interactions between us humans and these objects remains far-reaching. In this paper, we present I'm-HOI, a monocular scheme to faithfully capture the 3D motions of both the human and object in a novel setting, using a minimal amount of RGB camera and object-mounted Inertial Measurement Unit (IMU). It combines general motion inference and category-aware refinement. For the former, we introduce a holistic human-object tracking method to fuse the IMU signals and the RGB stream and progressively recover the human motions and subsequently the companion object motions. For the latter, we tailor a category-aware motion diffusion model, which is conditioned on both the raw IMU observations and the results from the previous stage under over-parameterization representation. It significantly refines the initial results and generates vivid body, hand, and object motions. Moreover, we contribute a large dataset with ground truth human and object motions, dense RGB inputs, and rich object-mounted IMU measurements. Extensive experiments demonstrate the effectiveness of I'm-HOI under a hybrid capture setting. Our dataset and code will be released to the community.

# Summary. An optional shortened abstract.
summary: In this paper, we present I'm-HOI, a monocular scheme to faithfully capture the 3D motions of both the human and object in a novel setting, using a minimal amount of RGB camera and object-mounted Inertial Measurement Unit (IMU).

tags: []

# Display this page in the Featured widget?
featured: true

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: 'https://arxiv.org/abs/2312.08869'
url_code: 'https://github.com/AfterJourney00/IMHD-Dataset'
url_dataset: 'https://github.com/AfterJourney00/IMHD-Dataset'
url_source: 'https://afterjourney00.github.io/IM-HOI.github.io/'
url_video: 'https://www.youtube.com/watch?v=MdG00uakBa8'

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# image:
#   caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/pLCdAaMFLTE)'
#   focal_point: ''
#   preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects:
  []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: ''
---

{{% callout note %}}
Click the _Cite_ button above to demo the feature to enable visitors to import publication metadata into their reference management software.
{{% /callout %}}

{{% callout note %}}
Create your slides in Markdown - click the _Slides_ button to check out the example.
{{% /callout %}}

Add the publication's **full text** or **supplementary notes** here. You can use rich formatting such as including [code, math, and images](https://docs.hugoblox.com/content/writing-markdown-latex/).
