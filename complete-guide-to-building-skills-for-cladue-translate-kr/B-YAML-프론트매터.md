# 참고 B: YAML 프론트매터(YAML Frontmatter)

## 필수 필드(Required fields)

```yaml
---
name: skill-name-in-kebab-case
description: 무엇을 하는지, 언제 사용하는지. 구체적인 트리거 문구를 포함합니다.
---
```

## 모든 선택 필드(All optional fields)

```yaml
name: skill-name
description: [필수 description]
license: MIT # 선택: 오픈소스용 라이선스
allowed-tools: "Bash(python:*) Bash(npm:*) WebFetch" # 선택: 도구 접근 제한
metadata: # 선택: 커스텀 필드
  author: Company Name
  version: 1.0.0
  mcp-server: server-name
  category: productivity
  tags: [project-management, automation]
  documentation: https://example.com/docs
  support: support@example.com
```

## 보안 참고(Security notes)

### 허용(Allowed):
- 모든 표준 YAML 타입 (문자열, 숫자, 불리언, 리스트, 객체)
- 커스텀 metadata 필드
- 긴 description (최대 1024자)

### 금지(Forbidden):
- XML 꺾쇠괄호 (`< >`) — 보안 제한
- YAML 내 코드 실행 (안전한 YAML 파싱을 사용)
- "claude" 또는 "anthropic" 접두사가 포함된 스킬명 (예약어)

---
> 원문: [**Anthropic - The Complete Guide to Building Skills for Claude.pdf**](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) from https://claude.com/blog/complete-guide-to-building-skills-for-claude
>
> 이 문서는 원문 PDF의 **한국어로 번역**한 것입니다.  원문 및 이미지의 저작권은 [Anthropic](https://www.anthropic.com/)에 있습니다. 
> 
> 번역에는 Claude Opus 4.6, ChatGPT 5.2 을 사용했습니다.