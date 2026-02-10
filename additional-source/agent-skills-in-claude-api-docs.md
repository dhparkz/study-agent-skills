# Agent Skills
> 원문 : https://platform.claude.com/docs/ko/agents-and-tools/agent-skills/overview.md


Agent Skills는 Claude의 기능을 확장하는 모듈식 역량입니다. 각 Skill은 관련성이 있을 때 Claude가 자동으로 사용하는 지침, 메타데이터, 선택적 리소스(스크립트, 템플릿)를 패키지로 제공합니다.

---

## Skills를 사용하는 이유

Skills는 Claude에게 도메인별 전문 지식을 제공하는 재사용 가능한 파일시스템 기반 리소스입니다: 워크플로, 컨텍스트, 모범 사례를 통해 범용 에이전트를 전문가로 변환합니다. 프롬프트(일회성 작업을 위한 대화 수준의 지침)와 달리, Skills는 필요에 따라 로드되며 여러 대화에서 동일한 안내를 반복적으로 제공할 필요를 없앱니다.

**주요 이점**:
- **Claude 전문화**: 도메인별 작업에 맞게 역량을 맞춤화
- **반복 감소**: 한 번 생성하면 자동으로 사용
- **역량 조합**: Skills를 결합하여 복잡한 워크플로 구축

<Note>
Agent Skills의 아키텍처와 실제 활용 사례에 대한 심층 분석은 엔지니어링 블로그를 참조하세요: [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills).
</Note>

## Skills 사용하기

Anthropic은 일반적인 문서 작업(PowerPoint, Excel, Word, PDF)을 위한 사전 구축된 Agent Skills를 제공하며, 사용자 정의 Skills도 직접 만들 수 있습니다. 둘 다 동일한 방식으로 작동합니다. Claude는 요청과 관련이 있을 때 자동으로 이를 사용합니다.

**사전 구축된 Agent Skills**는 claude.ai 및 Claude API의 모든 사용자에게 제공됩니다. 전체 목록은 아래의 [사용 가능한 Skills](#available-skills) 섹션을 참조하세요.

**사용자 정의 Skills**를 사용하면 도메인 전문 지식과 조직 지식을 패키지화할 수 있습니다. Claude의 제품 전반에서 사용할 수 있습니다: Claude Code에서 생성하거나, API를 통해 업로드하거나, claude.ai 설정에서 추가할 수 있습니다.

<Note>

**시작하기:**

- 사전 구축된 Agent Skills의 경우: [빠른 시작 튜토리얼](https://platform.claude.com/docs/ko/agents-and-tools/agent-skills/quickstart)을 참조하여 API에서 PowerPoint, Excel, Word, PDF Skills 사용을 시작하세요

- 사용자 정의 Skills의 경우: [Agent Skills Cookbook](https://platform.claude.com/cookbook/skills-notebooks-01-skills-introduction)을 참조하여 자체 Skills를 만드는 방법을 알아보세요
</Note>

## Skills 작동 방식

Skills는 Claude의 VM 환경을 활용하여 프롬프트만으로는 불가능한 역량을 제공합니다. Claude는 파일시스템 접근이 가능한 가상 머신에서 작동하므로, Skills는 지침, 실행 가능한 코드, 참조 자료를 포함하는 디렉토리로 존재할 수 있으며, 새 팀원을 위해 만드는 온보딩 가이드처럼 구성됩니다.

이 파일시스템 기반 아키텍처는 **점진적 공개**를 가능하게 합니다: Claude는 컨텍스트를 미리 소비하는 대신 필요에 따라 단계적으로 정보를 로드합니다.

### 세 가지 유형의 Skill 콘텐츠, 세 가지 수준의 로딩

Skills는 세 가지 유형의 콘텐츠를 포함할 수 있으며, 각각 다른 시점에 로드됩니다:

### 레벨 1: 메타데이터 (항상 로드됨)

**콘텐츠 유형: 지침**. Skill의 YAML 프론트매터는 검색 정보를 제공합니다:

```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
```

Claude는 시작 시 이 메타데이터를 로드하고 시스템 프롬프트에 포함합니다. 이 경량 접근 방식은 컨텍스트 페널티 없이 많은 Skills를 설치할 수 있음을 의미합니다; Claude는 각 Skill이 존재한다는 것과 언제 사용해야 하는지만 알고 있습니다.

### 레벨 2: 지침 (트리거 시 로드됨)

**콘텐츠 유형: 지침**. SKILL.md의 본문에는 절차적 지식이 포함됩니다: 워크플로, 모범 사례, 안내:

````markdown
# PDF Processing

## Quick start

Use pdfplumber to extract text from PDFs:

```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For advanced form filling, see [FORMS.md](FORMS.md).
````

Skill의 설명과 일치하는 요청을 하면, Claude는 bash를 통해 파일시스템에서 SKILL.md를 읽습니다. 그때서야 이 콘텐츠가 컨텍스트 윈도우에 들어갑니다.

### 레벨 3: 리소스 및 코드 (필요 시 로드됨)

**콘텐츠 유형: 지침, 코드, 리소스**. Skills는 추가 자료를 번들로 포함할 수 있습니다:

```
pdf-skill/
├── SKILL.md (main instructions)
├── FORMS.md (form-filling guide)
├── REFERENCE.md (detailed API reference)
└── scripts/
    └── fill_form.py (utility script)
```

**지침**: 전문화된 안내와 워크플로를 포함하는 추가 마크다운 파일(FORMS.md, REFERENCE.md)

**코드**: Claude가 bash를 통해 실행하는 실행 가능한 스크립트(fill_form.py, validate.py); 스크립트는 컨텍스트를 소비하지 않고 결정론적 작업을 제공합니다

**리소스**: 데이터베이스 스키마, API 문서, 템플릿, 예제와 같은 참조 자료

Claude는 참조될 때만 이러한 파일에 접근합니다. 파일시스템 모델은 각 콘텐츠 유형이 서로 다른 강점을 가짐을 의미합니다: 유연한 안내를 위한 지침, 신뢰성을 위한 코드, 사실 조회를 위한 리소스.

| 레벨 | 로드 시점 | 토큰 비용 | 콘텐츠 |
|-------|------------|------------|---------|
| **레벨 1: 메타데이터** | 항상 (시작 시) | Skill당 ~100 토큰 | YAML 프론트매터의 `name` 및 `description` |
| **레벨 2: 지침** | Skill 트리거 시 | 5k 토큰 미만 | 지침과 안내가 포함된 SKILL.md 본문 |
| **레벨 3+: 리소스** | 필요 시 | 사실상 무제한 | 콘텐츠를 컨텍스트에 로드하지 않고 bash를 통해 실행되는 번들 파일 |

점진적 공개는 주어진 시점에 관련 콘텐츠만 컨텍스트 윈도우를 차지하도록 보장합니다.

### Skills 아키텍처

Skills는 Claude가 파일시스템 접근, bash 명령, 코드 실행 역량을 갖춘 코드 실행 환경에서 실행됩니다. 이렇게 생각하면 됩니다: Skills는 가상 머신의 디렉토리로 존재하며, Claude는 컴퓨터에서 파일을 탐색할 때 사용하는 것과 동일한 bash 명령을 사용하여 상호작용합니다.

![Agent Skills 아키텍처 - Skills가 에이전트의 구성 및 가상 머신과 통합되는 방식을 보여줌](images/agent-skills-architecture.png)

**Claude가 Skill 콘텐츠에 접근하는 방식:**

Skill이 트리거되면, Claude는 bash를 사용하여 파일시스템에서 SKILL.md를 읽어 지침을 컨텍스트 윈도우로 가져옵니다. 해당 지침이 다른 파일(FORMS.md나 데이터베이스 스키마 등)을 참조하면, Claude는 추가 bash 명령을 사용하여 해당 파일도 읽습니다. 지침에서 실행 가능한 스크립트를 언급하면, Claude는 bash를 통해 실행하고 출력만 받습니다(스크립트 코드 자체는 컨텍스트에 들어가지 않습니다).

**이 아키텍처가 가능하게 하는 것:**

**온디맨드 파일 접근**: Claude는 각 특정 작업에 필요한 파일만 읽습니다. Skill에 수십 개의 참조 파일이 포함될 수 있지만, 작업에 영업 스키마만 필요하면 Claude는 해당 파일 하나만 로드합니다. 나머지는 파일시스템에 남아 토큰을 전혀 소비하지 않습니다.

**효율적인 스크립트 실행**: Claude가 `validate_form.py`를 실행할 때, 스크립트의 코드는 컨텍스트 윈도우에 로드되지 않습니다. 스크립트의 출력("Validation passed" 또는 특정 오류 메시지 등)만 토큰을 소비합니다. 이는 Claude가 즉석에서 동등한 코드를 생성하는 것보다 스크립트가 훨씬 더 효율적임을 의미합니다.

**번들 콘텐츠에 대한 실질적 제한 없음**: 파일은 접근될 때까지 컨텍스트를 소비하지 않으므로, Skills에 포괄적인 API 문서, 대규모 데이터셋, 광범위한 예제 또는 필요한 모든 참조 자료를 포함할 수 있습니다. 사용되지 않는 번들 콘텐츠에 대한 컨텍스트 페널티는 없습니다.

이 파일시스템 기반 모델이 점진적 공개를 가능하게 합니다. Claude는 온보딩 가이드의 특정 섹션을 참조하듯이 Skill을 탐색하며, 각 작업에 필요한 것만 정확히 접근합니다.

### 예시: PDF 처리 Skill 로딩

Claude가 PDF 처리 Skill을 로드하고 사용하는 방법은 다음과 같습니다:

1. **시작**: 시스템 프롬프트에 포함: `PDF Processing - Extract text and tables from PDF files, fill forms, merge documents`
2. **사용자 요청**: "이 PDF에서 텍스트를 추출하고 요약해 주세요"
3. **Claude 호출**: `bash: read pdf-skill/SKILL.md` → 지침이 컨텍스트에 로드됨
4. **Claude 판단**: 양식 작성이 필요하지 않으므로 FORMS.md를 읽지 않음
5. **Claude 실행**: SKILL.md의 지침을 사용하여 작업 완료

![컨텍스트 윈도우에 로드되는 Skills - Skill 메타데이터와 콘텐츠의 점진적 로딩을 보여줌](images/agent-skills-context-window.png)

다이어그램은 다음을 보여줍니다:
1. 시스템 프롬프트와 Skill 메타데이터가 사전 로드된 기본 상태
2. Claude가 bash를 통해 SKILL.md를 읽어 Skill을 트리거
3. Claude가 필요에 따라 FORMS.md와 같은 추가 번들 파일을 선택적으로 읽음
4. Claude가 작업을 진행

이 동적 로딩은 관련 Skill 콘텐츠만 컨텍스트 윈도우를 차지하도록 보장합니다.

## Skills가 작동하는 곳

Skills는 Claude의 에이전트 제품 전반에서 사용할 수 있습니다:

### Claude API

Claude API는 사전 구축된 Agent Skills와 사용자 정의 Skills를 모두 지원합니다. 둘 다 동일하게 작동합니다: 코드 실행 도구와 함께 `container` 매개변수에 관련 `skill_id`를 지정합니다.

**전제 조건**: API를 통해 Skills를 사용하려면 세 가지 베타 헤더가 필요합니다:
- `code-execution-2025-08-25` - Skills는 코드 실행 컨테이너에서 실행됩니다
- `skills-2025-10-02` - Skills 기능을 활성화합니다
- `files-api-2025-04-14` - 컨테이너에 파일을 업로드/다운로드하는 데 필요합니다

사전 구축된 Agent Skills는 `skill_id`(예: `pptx`, `xlsx`)를 참조하여 사용하거나, Skills API(`/v1/skills` 엔드포인트)를 통해 직접 만들어 업로드할 수 있습니다. 사용자 정의 Skills는 조직 전체에서 공유됩니다.

자세한 내용은 [Claude API에서 Skills 사용하기](https://platform.claude.com/docs/ko/build-with-claude/skills-guidee)를 참조하세요.

### Claude Code

[Claude Code](https://code.claude.com/docs/ko/overview)는 사용자 정의 Skills만 지원합니다.

**사용자 정의 Skills**: SKILL.md 파일이 포함된 디렉토리로 Skills를 생성합니다. Claude가 자동으로 검색하여 사용합니다.

Claude Code의 사용자 정의 Skills는 파일시스템 기반이며 API 업로드가 필요하지 않습니다.

자세한 내용은 [Claude Code에서 Skills 사용하기](https://code.claude.com/docs/ko/skills)를 참조하세요.

### Claude Agent SDK

[Claude Agent SDK](https://platform.claude.com/docs/ko/agent-sdk/overview)는 파일시스템 기반 구성을 통해 사용자 정의 Skills를 지원합니다.

**사용자 정의 Skills**: `.claude/skills/`에 SKILL.md 파일이 포함된 디렉토리로 Skills를 생성합니다. `allowed_tools` 구성에 `"Skill"`을 포함하여 Skills를 활성화합니다.

Agent SDK의 Skills는 SDK가 실행될 때 자동으로 검색됩니다.

자세한 내용은 [SDK의 Agent Skills](https://platform.claude.com/docs/ko/agent-sdk/skills)를 참조하세요.

### Claude.ai

[Claude.ai](https://claude.ai)는 사전 구축된 Agent Skills와 사용자 정의 Skills를 모두 지원합니다.

**사전 구축된 Agent Skills**: 이러한 Skills는 문서를 생성할 때 이미 백그라운드에서 작동하고 있습니다. Claude는 별도의 설정 없이 이를 사용합니다.

**사용자 정의 Skills**: 설정 > 기능을 통해 zip 파일로 자체 Skills를 업로드합니다. 코드 실행이 활성화된 Pro, Max, Team, Enterprise 플랜에서 사용할 수 있습니다. 사용자 정의 Skills는 각 사용자에게 개별적이며, 조직 전체에서 공유되지 않고 관리자가 중앙에서 관리할 수 없습니다.

Claude.ai에서 Skills를 사용하는 방법에 대한 자세한 내용은 Claude 도움말 센터의 다음 리소스를 참조하세요:
- [Skills란 무엇인가요?](https://support.claude.com/ko/articles/12512176-what-are-skills)
- [Claude에서 Skills 사용하기](https://support.claude.com/ko/articles/12512180-using-skills-in-claude)
- [사용자 정의 Skills 만드는 방법](https://support.claude.com/ko/articles/12512198-creating-custom-skills)
- [Skills를 사용하여 Claude에게 작업 방식 가르치기](https://support.claude.com/ko/articles/12580051-teach-claude-your-way-of-working-using-skills)

## Skill 구조

모든 Skill에는 YAML 프론트매터가 포함된 `SKILL.md` 파일이 필요합니다:

```yaml
---
name: your-skill-name
description: Brief description of what this Skill does and when to use it
---

# Your Skill Name

## Instructions
[Clear, step-by-step guidance for Claude to follow]

## Examples
[Concrete examples of using this Skill]
```

**필수 필드**: `name` 및 `description`

**필드 요구사항**:

`name`:
- 최대 64자
- 소문자, 숫자, 하이픈만 포함 가능
- XML 태그를 포함할 수 없음
- 예약어를 포함할 수 없음: "anthropic", "claude"

`description`:
- 비어 있으면 안 됨
- 최대 1024자
- XML 태그를 포함할 수 없음

`description`에는 Skill이 하는 일과 Claude가 언제 사용해야 하는지를 모두 포함해야 합니다. 완전한 작성 안내는 [모범 사례 가이드](/docs/ko/agents-and-tools/agent-skills/best-practices)를 참조하세요.

## 보안 고려사항

Skills는 직접 만들었거나 Anthropic에서 제공한 신뢰할 수 있는 소스의 것만 사용할 것을 강력히 권장합니다. Skills는 지침과 코드를 통해 Claude에게 새로운 역량을 제공하며, 이것이 강력한 이유이기도 하지만, 악의적인 Skill이 Claude에게 Skill의 명시된 목적과 일치하지 않는 방식으로 도구를 호출하거나 코드를 실행하도록 지시할 수 있음을 의미합니다.

<Warning>
신뢰할 수 없거나 알 수 없는 소스의 Skill을 사용해야 하는 경우, 극도의 주의를 기울이고 사용 전에 철저히 감사하세요. Skill 실행 시 Claude가 가진 접근 권한에 따라, 악의적인 Skills는 데이터 유출, 무단 시스템 접근 또는 기타 보안 위험을 초래할 수 있습니다.
</Warning>

**주요 보안 고려사항**:
- **철저한 감사**: Skill에 번들된 모든 파일을 검토하세요: SKILL.md, 스크립트, 이미지, 기타 리소스. 예상치 못한 네트워크 호출, 파일 접근 패턴, Skill의 명시된 목적과 일치하지 않는 작업과 같은 비정상적인 패턴을 찾으세요
- **외부 소스는 위험**: 외부 URL에서 데이터를 가져오는 Skills는 특히 위험합니다. 가져온 콘텐츠에 악의적인 지침이 포함될 수 있기 때문입니다. 신뢰할 수 있는 Skills도 외부 종속성이 시간이 지남에 따라 변경되면 손상될 수 있습니다
- **도구 오용**: 악의적인 Skills는 도구(파일 작업, bash 명령, 코드 실행)를 해로운 방식으로 호출할 수 있습니다
- **데이터 노출**: 민감한 데이터에 접근할 수 있는 Skills는 외부 시스템으로 정보를 유출하도록 설계될 수 있습니다
- **소프트웨어 설치처럼 취급**: 신뢰할 수 있는 소스의 Skills만 사용하세요. 민감한 데이터나 중요한 작업에 접근할 수 있는 프로덕션 시스템에 Skills를 통합할 때 특히 주의하세요

## 사용 가능한 Skills

### 사전 구축된 Agent Skills

다음 사전 구축된 Agent Skills를 즉시 사용할 수 있습니다:

- **PowerPoint (pptx)**: 프레젠테이션 생성, 슬라이드 편집, 프레젠테이션 콘텐츠 분석
- **Excel (xlsx)**: 스프레드시트 생성, 데이터 분석, 차트가 포함된 보고서 생성
- **Word (docx)**: 문서 생성, 콘텐츠 편집, 텍스트 서식 지정
- **PDF (pdf)**: 서식이 지정된 PDF 문서 및 보고서 생성

이러한 Skills는 Claude API와 claude.ai에서 사용할 수 있습니다. API에서 사용을 시작하려면 [빠른 시작 튜토리얼](/docs/ko/agents-and-tools/agent-skills/quickstart)을 참조하세요.

### 사용자 정의 Skills 예시

사용자 정의 Skills의 전체 예시는 [Skills cookbook](https://platform.claude.com/cookbook/skills-notebooks-01-skills-introduction)을 참조하세요.

## 제한사항 및 제약조건

이러한 제한사항을 이해하면 Skills 배포를 효과적으로 계획하는 데 도움이 됩니다.

### 크로스 서피스 가용성

**사용자 정의 Skills는 서피스 간에 동기화되지 않습니다**. 한 서피스에 업로드된 Skills는 다른 서피스에서 자동으로 사용할 수 없습니다:

- Claude.ai에 업로드된 Skills는 API에 별도로 업로드해야 합니다
- API를 통해 업로드된 Skills는 Claude.ai에서 사용할 수 없습니다
- Claude Code Skills는 파일시스템 기반이며 Claude.ai와 API 모두와 별개입니다

사용하려는 각 서피스에 대해 Skills를 별도로 관리하고 업로드해야 합니다.

### 공유 범위

Skills는 사용하는 곳에 따라 다른 공유 모델을 가집니다:
- **Claude.ai**: 개별 사용자만; 각 팀원이 별도로 업로드해야 합니다
- **Claude API**: 워크스페이스 전체; 모든 워크스페이스 멤버가 업로드된 Skills에 접근할 수 있습니다
- **Claude Code**: 개인(`~/.claude/skills/`) 또는 프로젝트 기반(`.claude/skills/`); Claude Code Plugins를 통해 공유할 수도 있습니다

Claude.ai는 현재 사용자 정의 Skills의 중앙 관리자 관리 또는 조직 전체 배포를 지원하지 않습니다.

### 런타임 환경 제약조건

Skill에서 사용할 수 있는 정확한 런타임 환경은 사용하는 제품 서피스에 따라 다릅니다.

 - **Claude.ai**:
    - **가변적 네트워크 접근**: 사용자/관리자 설정에 따라 Skills는 전체, 부분 또는 네트워크 접근이 없을 수 있습니다. 자세한 내용은 [파일 생성 및 편집](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_6b7e833898) 지원 문서를 참조하세요.
- **Claude API**:
    - **네트워크 접근 없음**: Skills는 외부 API 호출이나 인터넷 접근을 할 수 없습니다
    - **런타임 패키지 설치 불가**: 사전 설치된 패키지만 사용할 수 있습니다. 실행 중에 새 패키지를 설치할 수 없습니다.
    - **사전 구성된 종속성만**: 사용 가능한 패키지 목록은 [코드 실행 도구 문서](/docs/ko/agents-and-tools/tool-use/code-execution-tool)를 확인하세요
- **Claude Code**:
    - **전체 네트워크 접근**: Skills는 사용자 컴퓨터의 다른 프로그램과 동일한 네트워크 접근 권한을 가집니다
    - **전역 패키지 설치 비권장**: Skills는 사용자 컴퓨터에 간섭하지 않도록 로컬에만 패키지를 설치해야 합니다

이러한 제약조건 내에서 작동하도록 Skills를 계획하세요.

## 다음 단계

- **[Agent Skills 시작하기](https://platform.claude.com/docs/ko/agents-and-tools/agent-skills/quickstart)** — 첫 번째 Skill 만들기
- **[API 가이드](https://platform.claude.com/docs/ko/build-with-claude/skills-guide)** — Claude API에서 Skills 사용하기
- **[Claude Code에서 Skills 사용하기](https://code.claude.com/docs/ko/skills)** — Claude Code에서 사용자 정의 Skills 생성 및 관리
- **[SDK에서 Skills 사용하기](https://platform.claude.com/docs/ko/agent-sdk/skills)** — TypeScript와 Python에서 프로그래밍 방식으로 Skills 사용
- **[작성 모범 사례](https://platform.claude.com/docs/ko/agents-and-tools/agent-skills/best-practices)** — Claude가 효과적으로 사용할 수 있는 Skills 작성하기