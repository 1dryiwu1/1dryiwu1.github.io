# Portfolio Redesign Guide

## 1. Project Goal

This website is not a general personal homepage. It is a job-facing portfolio site that will be sent to HR and interviewers.

Its core goal is:

- attract HR in the first screen
- help HR quickly understand the candidate's value
- give technical interviewers enough depth to continue asking questions
- present the candidate as a structured, credible, engineering-oriented person

The priority order is:

1. easy for HR to understand
2. visually memorable and polished
3. technically convincing

This means the site should never become a pure visual showpiece, a game-style promo page, or a self-indulgent technical archive.

## 2. Core Design Direction

The chosen global direction is:

`industrial dossier interface`

More specifically:

- industrial sci-fi restraint
- dossier-like information hierarchy
- engineering credibility
- calm system-running atmosphere

The site should feel like:

`a carefully designed technical personal archive`

It should not feel like:

- a flashy cyberpunk template
- a generic student portfolio
- a game landing page clone
- a neon-heavy hacker page

## 3. Audience Strategy

The site serves two readers:

### HR

HR needs to understand very quickly:

- who this candidate is
- what they have built
- what they are strong at
- whether they are worth moving forward

So the site must be:

- easy to scan
- easy to retell
- low-friction to understand

### Technical interviewers

Technical interviewers need signals that the candidate has:

- real project experience
- systems thinking
- implementation depth
- engineering communication ability

So the site must still preserve:

- technically meaningful project descriptions
- coherent ability structure
- credible wording

## 4. Global Style Rules

### 4.1 Mood

Global mood:

- calm
- precise
- controlled
- technical
- premium

The page should feel "online" and "running", but not busy.

### 4.2 Color

Color distribution is fixed:

- background: deep graphite gray, not pure black
- primary text: cold off-white
- secondary text: cool medium gray
- accent green: only 30% to 40% of visual emphasis
- metal tones: dark silver, graphite metal, cool gray

Green is a functional state color, not the dominant identity color.

Use green for:

- status lines
- active indicators
- system highlights
- thin energy seams
- scan accents

Do not use green as:

- full-page glow wash
- large block fill everywhere
- always-on aggressive border lighting

### 4.3 Material

The first screen visual language should be led by:

- brushed metal
- frosted glass
- segmented panels
- seams and lock points
- subtle reflective edges

This metal-first approach should influence later sections too, even if later sections are visually lighter.

### 4.4 Typography

Typography must serve clarity first.

- main Chinese headings: stable, clean, weighted
- English supporting labels: structured, technical, system-like
- body copy: short paragraphs, easy scan, never over-written
- titles should reduce in drama after the first screen

Do not use:

- overly decorative fonts
- overly compressed futuristic display fonts
- huge amounts of letter-spaced text everywhere
- long blocks of emotional self-description

## 5. Motion Rules

Motion must follow one philosophy:

`device feedback over visual performance`

### Allowed motion

- fade-in reveals
- slight position shifts
- low-frequency pulses
- scan passes
- highlight responses
- subtle system wake-up cues

### Forbidden motion

- frequent flashing
- decorative constant rotation
- bouncing icons
- large amounts of particle drift
- continuous motion in multiple unrelated elements
- text that keeps moving after it has entered

### Motion density by screen

- Screen 1: strongest motion presence, still restrained
- Screen 2: medium, information-first
- Screen 3: low-medium
- Screen 4-6: low

Text should remain stable almost everywhere.

## 6. Information Architecture

The whole site is structured as six screens.

### Screen 1

Role:

- attract attention
- define identity
- establish the overall world and tone

### Screen 2

Role:

- prove capability through projects
- help HR quickly build trust

### Screen 3

Role:

- clarify ability structure
- show the candidate is focused, not scattered

### Screen 4

Role:

- show growth continuity
- make the profile feel earned, not accidental

### Screen 5

Role:

- add personal dimension
- show expression, discipline, and collaboration traits

### Screen 6

Role:

- provide clear contact exit
- end the reading experience cleanly

## 7. Screen 1: Hero Direction

### 7.1 Hero concept

The first screen is fully defined as:

`an industrial dossier homepage`

This is a text-led hero with a supporting right-side industrial core visual.

### 7.2 Layout

Layout ratio:

- left side: 55%
- right side: 45%

The hero is text-dominant.

The right-side visual should support and anchor the mood, not overpower the candidate identity.

### 7.3 Hero copy structure

Main title:

- Chinese full name

Supporting title:

- full pinyin name

Identity line:

`区块链开发者 / 分布式系统实践 / 智能合约工程`

Summary:

`专注于将链上逻辑、系统设计与工程实现落到可运行的项目中。`

Bottom system strip:

- `PROJECTS // 03`
- `RANK // TOP 5%`
- `FOCUS // RAFT · SOLIDITY · VUE`
- `LOCATION // CHONGQING`

### 7.4 Hero visual

The right-side visual is fixed as:

`vertical metallic core chamber`

Requirements:

- complete chamber body as the main silhouette
- some exposed internal structure
- metallic shell as the dominant material
- green only in slits, status seams, and internal signals
- slight right-edge cropping is allowed
- the object should feel like a real industrial control component

It should not look like:

- a glowing pillar
- a logo
- a sci-fi ornament
- a fantasy relic

### 7.5 Hero motion

Hero motion rules:

- internal low-frequency pulse
- occasional scan pass
- tiny calibration movement in outer structures
- subtle path signal

Text stays still.

## 8. Screen 2: Project Evidence Section

### 8.1 Section role

This section exists for HR first.

It must answer:

- what has this person built
- what is the strongest project
- what kind of technical direction do they have

### 8.2 Section title

Title:

`核心项目经历`

Support line:

`围绕分布式系统、智能合约与工程实现展开的代表性项目。`

### 8.3 Layout

The chosen structure is:

- 1 primary project
- 2 secondary projects

This is the best balance between clarity and depth for HR-facing reading.

### 8.4 Primary project

Primary project:

`MedTrust-Raft`

Primary positioning sentence:

`面向多医疗机构可信数据共享场景的联盟治理平台。`

Technical sub-line:

`基于 Raft 共识机制构建核心协作逻辑。`

Role line:

`主导系统设计，并独立完成 Raft 核心机制实现。`

Key points:

1. `从零实现 Raft 核心流程，覆盖 Leader 选举、日志复制与心跳维持等关键机制。`
2. `围绕多机构协作场景设计联盟治理架构，提升医疗数据共享过程中的可信性与一致性。`
3. `完成从共识层到服务层的整体工程串联，使系统具备可运行、可演示的完整形态。`

Tech line:

`Go / Raft / Gin / BoltDB / Vue 3`

### 8.5 Secondary project 01

Project:

`公益众筹智能合约`

Positioning:

`基于以太坊的去中心化众筹合约实践。`

Role:

`负责合约逻辑设计与关键安全机制实现。`

Key points:

1. `围绕众筹流程实现目标验证、自动拨款与失败退款等核心逻辑。`
2. `按里程碑拆分资金释放流程，增强资金使用的可控性与透明度。`
3. `在合约设计中引入常见安全模式，降低重入等链上风险。`

Tech line:

`Solidity / Ethereum / Truffle / Ganache`

### 8.6 Secondary project 02

Project:

`AI 工具与技术文档实践`

Positioning:

`将 AI 工具、需求分析与技术文档能力结合到开发流程中的持续实践。`

Role:

`负责方案整理、技术表达与辅助研发流程优化。`

Key points:

1. `能够将复杂需求快速拆解为结构化方案，提高沟通与执行效率。`
2. `在项目过程中持续输出需求分析、技术方案与架构说明文档。`
3. `熟悉借助 AI 工具辅助信息整理、内容生成与开发支持，提升个人交付效率。`

Tech line:

`AI Tools / Technical Writing / Solution Design`

## 9. Screen 3: Skills and Capability Structure

### 9.1 Section role

This section should not be a badge wall.

Its job is to show:

- the candidate has a clear technical center of gravity
- the capability stack is structured
- the projects and skills reinforce each other

### 9.2 Title

`技术栈与能力`

Support line:

`围绕分布式系统、链上开发与工程实现形成的能力结构。`

### 9.3 Core capability pillars

Three high-priority pillars:

1. `分布式系统`
   - `Raft / 一致性机制 / 服务协作 / 系统设计`

2. `区块链开发`
   - `Solidity / Ethereum / 智能合约 / 链上逻辑`

3. `工程实现`
   - `Go / Vue 3 / Gin / BoltDB / 项目落地`

### 9.4 Skill groups

Grouped skill list:

- `Languages`
  - `Go / Solidity / Python / JavaScript`

- `Backend & Infra`
  - `Gin / BoltDB / 共识机制实现 / 服务架构`

- `Frontend`
  - `Vue 3 / 交互实现 / 页面工程化`

- `Blockchain`
  - `Ethereum / Truffle / Ganache / 智能合约开发`

### 9.5 Working style support line

`具备需求分析、技术方案整理与文档输出能力，能够借助 AI 工具辅助信息处理与开发流程。`

## 10. Screen 4: Growth Timeline

### 10.1 Section role

This section shows continuity.

It tells HR:

- the profile has a growth path
- the candidate did not appear from nowhere with random projects
- technical ability has developed through stages

### 10.2 Title

`成长时间线`

Support line:

`从专业学习到项目实践，逐步形成面向系统与工程落地的能力方向。`

### 10.3 Timeline stages

1. `专业起点`
   - 进入区块链工程相关学习，建立对分布式系统与链上开发的基础认知。

2. `算法与竞赛训练`
   - 通过比赛和系统性训练补强问题拆解、逻辑推演和实现能力。

3. `项目实践阶段`
   - 开始把学习内容落到真实项目中，重点进入联盟治理、共识机制、智能合约等更成体系的实践。

4. `工程化表达与能力收束`
   - 不再只是做功能，而是开始关注架构、方案、文档、协作和交付质量，形成更完整的工程思维。

## 11. Screen 5: Outside Work

### 11.1 Section role

This section should not become a hobby gallery.

Its purpose is to show:

- observation
- rhythm
- expression

These are personality signals that support employability.

### 11.2 Title

`工作之外`

Support line:

`保持创造力、表达能力与长期学习节奏的个人侧面。`

### 11.3 Content blocks

#### 绘画

`通过线条、结构与色彩训练观察力，也让自己在技术工作之外保留视觉表达与创造性的出口。`

#### 吉他

`把音乐当作调节节奏与保持专注的方式，长期的练习过程也影响了我对结构、节奏与耐心的理解。`

#### 文档与写作

`习惯把需求、方案与技术决策记录清楚，让复杂问题更容易被沟通、协作与复盘。`

## 12. Screen 6: Contact and Closing

### 12.1 Section role

This is not a feature zone.

It is the clean ending point and contact handoff.

### 12.2 Title

`联系我`

Support line:

`如果你希望进一步了解项目、经历或合作可能，欢迎直接联系。`

### 12.3 Contact priority

Display order:

1. email
2. phone
3. GitHub

Optional later addition:

- `Resume`

Do not use:

- a complex form
- many social icons
- loud CTA buttons
- over-marketing copy

## 13. Visual Weight by Section

This distribution is fixed.

### High visual weight

- Screen 1 only

### Medium visual weight

- Screen 2
- Screen 3

### Lower visual weight

- Screen 4
- Screen 5
- Screen 6

This is essential to avoid every screen competing for attention.

## 14. What Not to Do

The redesign should explicitly avoid:

- pure black background
- overuse of green glow
- random floating particles dominating the scene
- every section having a big visual gimmick
- cute animated hobby icons
- game promo page imitation
- overlong copy blocks
- jargon-first writing that HR cannot parse quickly
- giant badge walls for skills
- repeated hover-light gimmicks
- visual noise that breaks the dossier feeling

## 15. Recommended Vue Structure

The site should be split by section responsibility.

Suggested top-level components:

- `HeroSection`
- `ProjectsSection`
- `SkillsSection`
- `TimelineSection`
- `OutsideWorkSection`
- `ContactSection`

Suggested shared components:

- `SectionHeader`
- `MetricStrip`
- `ProjectCard`
- `TimelineItem`
- `InfoBlock`

## 16. Data Structure Strategy

Content-heavy sections should be data-driven instead of fully hardcoded in the template.

Recommended data arrays:

- hero metrics
- project list
- capability groups
- timeline items
- outside-work items
- contact items

This makes later content editing much easier and safer.

## 17. Motion Implementation Strategy

Recommended motion organization:

- one shared reveal system for scroll-based entrances
- one dedicated hero motion controller
- section-local hover logic only where truly needed
- no disconnected motion systems per section

If GSAP remains in use:

- use it primarily for hero system motion
- use it secondarily for reveal choreography
- avoid using GSAP to animate every small decorative detail

## 18. Build Order Recommendation

When implementing the redesign, use this order:

1. rebuild Screen 1
2. rebuild Screen 2
3. rebuild Screen 3
4. rebuild Screen 4
5. rebuild Screen 5
6. rebuild Screen 6
7. unify colors, spacing, and motion
8. do final mobile adjustment
9. update static export for GitHub Pages

This order ensures the most valuable interview-facing sections are solved first.

## 19. Final Standard

The finished site should make HR feel:

- this person is serious
- this person has real projects
- this person communicates clearly
- this person has taste

It should make technical interviewers feel:

- the candidate has systems awareness
- the projects are credible
- the stack is coherent
- the site is not compensating for lack of substance

If the redesign succeeds, the final impression should be:

`clear enough for HR, strong enough for engineers, refined enough to be remembered`
